# Symptoms, causes and checks

Each row: what the user sees, the likely causes in the order to check them, and the tool or
docs page that confirms it. Run the checks; do not tell the user a cause you have not seen.

## Symptom → cause → check

| Symptom                               | Likely cause                                          | Check                                                                                                       | Fix                                                                                                                    |
| ------------------------------------- | ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| One sale did not show up              | The sale never reached OneClick                       | `countMetaTrackingRawLogs` / `getMetaTrackingRawLogs` for the platform and the day; match by date and value | No raw log: see "Integration stopped receiving"                                                                        |
|                                       | It arrived but did not become a conversion            | Raw log with `metaTrackingStructuredId` null; read the payload's status, stage or product                   | Explain the rule that excluded it (per-platform notes below); a CRM stage not selected → `setTrackingConversionStages` |
|                                       | It became a conversion but has no campaign            | Structured log whose `origin` carries no Meta ids                                                           | "Sales not attributed" below                                                                                           |
|                                       | Hotmart refused it (new token on Hotmart)             | Card warning in the app: "A Hotmart está enviando um token diferente do que aprendemos"                     | User clicks **Reaprender token** in the app, only if they changed the token; Hotmart retries refused deliveries        |
| Sales not attributed to a campaign/ad | Ad without UTMs, or UTMs without ids                  | `listCampaignAdsFromMeta` → `getCreativeCopyFromMeta` → `urlTags` of the ads that sold                      | Ads carry the UTM parameters the integration gave, with at least `{{ad.id}}`                                           |
|                                       | UTMs lost between landing page and checkout           | `listTrackingIntegrations` → `scriptDePassagemStatus`                                                       | Pass-through script on every sales page (site header preferred, or Google Tag Manager)                                 |
|                                       | Ids truncated by long names                           | `urlTags` puts names before ids                                                                             | Put the id first; Meta limits the length                                                                               |
|                                       | CRM card has no UTMs (form does not fill them)        | Raw payload of a lead with empty UTM fields                                                                 | The CRM's form must carry the UTMs (platform docs page, e.g. Pipedrive's custom UTM fields)                            |
| ROAS or revenue looks wrong           | Wrong default currency                                | `listTrackingIntegrations` / `updateTrackingIntegrationSettings` result `settings`                          | `updateTrackingIntegrationSettings` `currency` (only if payloads do not carry one; triggers reprocessing)              |
|                                       | Order bumps / upsells sent on the same webhook        | Structured logs `products` per sale                                                                         | User decides which products the platform sends (Hotmart docs suggest the main product only)                            |
|                                       | Same campaign selling on two platforms                | Structured logs of two platforms with the same `origin` campaign                                            | Docs recommend one platform per campaign                                                                               |
|                                       | Comparing with Ads Manager's ROAS                     | `getCampaignLiveMetaInsights` (`roas` from Meta's `purchase_roas`) vs `getMetaAdAccountTrackingInsights`    | Two measurements; explain (`en/oneclick-tracking/pixel-vs-webhook-and-utm`)                                            |
|                                       | Optimization reads a blend                            | Integration's `attributionMode`                                                                             | Explain the mode (`en/oneclick-tracking/attribution-modes`); change only on request                                    |
| Integration stopped receiving         | Credential refused                                    | `credentialStatus` rejected                                                                                 | `reconnectTrackingIntegration` with a new token from the user                                                          |
|                                       | Webhook deleted or changed on the platform            | `webhookStatus` inactive; no raw logs since a date                                                          | `getTrackingWebhookUrl` → user re-registers it with the events the platform page names                                 |
|                                       | Simply no sales in the period                         | Platform's own sales report for the same days                                                               | Nothing to fix                                                                                                         |
| CRM stages not counted                | No stages chosen                                      | `conversionStagesStatus` missing                                                                            | `listCrmFunnelStages` → `setTrackingConversionStages`                                                                  |
|                                       | Stage or funnel deleted/renamed on the CRM            | `listCrmFunnelStages` `selection` vs current funnels; `reconnectTrackingIntegration` returns both           | `setTrackingConversionStages` with the whole selection                                                                 |
|                                       | Deal moved to a stage not selected                    | Raw payload's stage                                                                                         | Add the stage, keeping every other funnel in the list                                                                  |
|                                       | Chatwoot label not in the list                        | Integration `settings` → conversion labels                                                                  | `updateTrackingIntegrationSettings` `conversionLabels` (case and spaces ignored; triggers reprocessing)                |
| Campaign name not updated             | No attributed conversion yet                          | Structured logs with `origin` ids                                                                           | Wait: naming starts after the first attributed conversion                                                              |
|                                       | Hourly cycle                                          | —                                                                                                           | Naming updates about once an hour; refresh Ads Manager and check its filters                                           |
|                                       | Campaign without optimization and no spend in 14 days | `getCampaignLiveMetaInsights` spend                                                                         | It keeps its last name                                                                                                 |
|                                       | `-` in some periods                                   | Period starts before the account's first tracked conversion                                                 | Fills in over time; "Maximum" stays `-` for campaigns older than that                                                  |

## UTMs

OneClick's default pattern (ids in every parameter):

```
utm_source=facebook-oneclick-ads&utm_content={{ad.id}}&utm_medium={{adset.id}}&utm_campaign={{campaign.id}}
```

- Keep at least `{{ad.id}}`: it also identifies the ad set and the campaign.
- A GA4 pattern with names can be combined with ids (hybrid patterns:
  `en/oneclick-tracking`, "UTM setup").
- UTMs go in the destination link or in the ad's URL parameters field; **every** ad needs
  them. Ads Manager edits several ads' URL parameters at once.
- A new creative does not inherit URL tags from a reference ad: new ads created without
  them lose attribution. Check new ads first when tracking drops after a launch.
- Use the parameters `createTrackingIntegration` returned for this account when the user
  still has them; do not retype them from memory.

## Per-platform notes (from the platform docs pages)

| Platform      | What counts / what to check                                                                                                                                                                                                          |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Hotmart       | Webhook in Ferramentas > Ver Todas > Webhooks, one per product, all purchase events checked. Token (hottok) learned from the first sale; a new token is refused until **Reaprender token** in the app.                               |
| Shopify       | Webhook event "Order Update", JSON. Only orders with payment status **Paid** count; pending, canceled, refunded do not. Value is the order total net of discounts, with shipping and taxes; each order counted once by order number. |
| Kommo         | Long-lived token (admin, up to 5 years) + workspace subdomain. Webhook event "Lead status changed". Conversion on the chosen stage.                                                                                                  |
| Pipedrive     | API token from Personal Preferences → API. Deals need custom UTM text fields filled by the form. Webhook: action change, object deal.                                                                                                |
| Chatwoot      | Conversion = the conversation labels configured; webhook triggers per the docs page.                                                                                                                                                 |
| Tintim        | Lead statuses have no funnel: one selection entry without `funnelId`.                                                                                                                                                                |
| Custom (Make) | Only for platforms with no native integration; the payload must carry the UTM fields or nothing can be attributed. Prefer a native integration when one now exists.                                                                  |

Platforms that need a credential (token): kommo, hubspot, pipedrive, clint, go_high_level,
pandachat, tintim, agendor, piperun, bitrix24, datacrazy, nectar_crm, bwcommerce,
loja_integrada. CRMs with stages to choose: kommo, pipedrive, hubspot, go_high_level, clint,
tintim, pandachat, agendor, piperun, bitrix24, datacrazy, nectar_crm, rd_station_crm.
