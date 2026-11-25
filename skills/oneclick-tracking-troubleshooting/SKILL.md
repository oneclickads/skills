---
name: oneclick-tracking-troubleshooting
description: Diagnose and fix OneClick Tracking problems through the OneClick Ads MCP — a sale that did not show up, sales not attributed to a campaign or ad, ROAS that looks wrong, an integration that stopped receiving, CRM stages not counted as conversions, missing or wrong UTMs — from the integration health and the tracking logs, then reconnecting a refused credential, choosing the CRM conversion stages or changing the integration settings after confirming. Use when the user says "minha venda não apareceu", "vendas sem campanha", "o ROAS está errado", "o rastreamento parou", "o CRM não conta conversão", "minhas UTMs estão certas?", "o nome da campanha não atualiza". Requires the OneClick Ads MCP connected (write scope for fixes).
---

# OneClick Tracking troubleshooting (OneClick Ads)

OneClick Tracking receives each sale from the platform that charged it (webhook) or from a
CRM (access credential) and attributes it to the Meta campaign, ad set and ad by the
**UTMs** that came with it. It does not send anything to Meta and does not replace the
pixel. Almost every problem sits in one link of that chain: the platform sends → OneClick
receives (raw log) → OneClick parses it into a conversion (structured log) → the UTMs carry
the Meta ids → the naming and the optimizations read it.

Find which link broke with read tools, fix what the MCP can fix, and send the user to
the app or their platform for the rest. First-time setup is the skill
`oneclick-ads-onboarding`.

Talk to the user in their language (usually pt-BR) and by **names** — ad accounts,
platforms, campaigns, funnels, stages. Never ask the user for an id and never invent one;
every id comes from a listing tool. If the client exposes only `search`/`describe`/`call`,
search tools in English ("tracking integrations", "tracking raw logs").

The docs are the source of truth for product behaviour. When a detail here is not enough,
read the page with `getDocsPage` (paths below) instead of guessing.

## Workflow

### 1. Account and integrations

1. `getMetaAdAccountsWithTracking` → the accounts with tracking. Several: ask which by
   name. None: tracking is not set up → skill `oneclick-ads-onboarding`.
2. `listTrackingIntegrations` for that account. It is the health check; read every field
   per integration:
   - `credentialStatus` **rejected** → the platform refuses the stored credential: its
     funnel stages cannot be read or changed, and where OneClick reads each sale through
     it (Agendor, DataCrazy, Bitrix24) conversions are dropped. Kommo and Pipedrive keep
     receiving conversions from stages already set. Fix first (step 3).
   - `conversionStagesStatus` **missing** → a CRM with no stages chosen records
     **nothing**, however healthy the rest looks: fix (step 3).
   - `webhookStatus` **inactive** → nothing has arrived and it is no longer new;
     **pending** → configured recently, nothing yet; **active** → payloads have arrived.
   - `scriptDePassagemStatus` → whether the pass-through script is tagging conversions
     with campaign/ad set/ad ids.
   - `utmStatus` + `utmStatusUpdatedAt` → the **last value recorded**, not a fresh check;
     for a current one send the user to the tracking page in the app.
   - The platform the user names is not listed → the sale cannot arrive; set it up with
     `oneclick-ads-onboarding`.

### 2. Follow the sale through the logs

Count before listing: `countMetaTrackingRawLogs` / `countMetaTrackingStructuredLogs`, then
`getMetaTrackingRawLogs` / `getMetaTrackingStructuredLogs` paged with `limit`/`offset`.
Always pass `adAccountId` (the value from `getMetaAdAccountsWithTracking`), the `platform`
and the smallest `datePreset` that covers the sale (boundaries in America/Sao_Paulo).

| What you find                                                      | Where it broke                                                                           |
| ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| No raw log for the period                                          | The platform is not sending: webhook missing, wrong URL or wrong events                  |
| Raw log with `metaTrackingStructuredId` null                       | Arrived but produced no conversion: read the payload (status, stage, product) to see why |
| Structured log whose `origin` has no campaign/ad set/ad ids        | Recorded but not attributable: the sale came without UTM ids                             |
| Structured log with `origin` ids, but `campaignId`/`adId` null     | Normal: those fields fill only once an optimization reads the entity — use `origin`      |
| Structured log with `origin` ids, but no naming in Ads Manager yet | Naming updates about once an hour, after the first attributed conversion                 |

Match a sale by `transactionId`, `orderDate` and `transactionValue` (buyer data is not
in the logs: ask for order number, date and value). Map `origin` ids to names with
`listCampaignsFromMeta` / `listCampaignAdsFromMeta`.

Symptom by symptom (sale missing, unattributed, ROAS off, integration stopped, CRM not
counting, UTMs, naming): [references/symptoms.md](references/symptoms.md).

### 3. Fix what the tools fix

Every fix writes on a live integration. State the change in plain words and get an
explicit yes before calling.

| Cause                                                                                                  | Tool                                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `credentialStatus` rejected, or the user regenerated a token                                           | `reconnectTrackingIntegration` with the new token the user generates. Ask for a token only then. Compare the returned `funnels` with `selection`: a selected funnel or stage that is gone stopped recording → `setTrackingConversionStages` |
| CRM stages missing or wrong                                                                            | `listCrmFunnelStages` → show funnel and stage **names** → `setTrackingConversionStages` with the **whole** selection (it replaces it; a funnel left out stops recording silently)                                                           |
| Currency, attribution mode, campaign-name indicators, Chatwoot labels, Yampi alias, account identifier | `updateTrackingIntegrationSettings`, a PATCH: send only what the user asked to change. Relay `changed`; when `reprocessing` is true, say recorded conversions are being re-read (a few minutes)                                             |
| Webhook missing or deleted on the platform                                                             | `getTrackingWebhookUrl` → the user pastes it on the platform, following the platform's docs page                                                                                                                                            |
| Ads without UTM ids                                                                                    | The user fixes the ads' URL parameters in Ads Manager (see references)                                                                                                                                                                      |

- Changing `attributionMode` or `campaignNameIndicators` renames running campaigns and
  changes the conversion numbers OneClick reports; the optimizations use the new mode from
  the next cycles on.
- There is **no reprocess on demand**. OneClick re-reads recorded conversions on its own
  only when the currency, the Chatwoot labels or the account identifier change.
- After a fix, check `listTrackingIntegrations`, then the logs after the next sale.

### 4. Report

In one short answer: which link broke, the evidence (counts, the sale's log), what you
changed and what the user still does on their platform or in the app.

## Rules

- Read before you write; never ask for a token before `listTrackingIntegrations` or the
  user gives a reason.
- Never call `deleteTrackingIntegration` to "fix" or "reset" anything: it erases the
  integration's conversion history for good. Only on an explicit request to remove it,
  after saying which platform, which account and that the history does not come back.
- Never retype or summarise a webhook URL, UTM parameters or script: hand them over in a
  code block exactly as a tool returned them.
- OneClick and Ads Manager numbers differ by design (pixel vs. sale at the source).
  Explain before calling it a defect.

## What you cannot do here

- Reprocess or resend past sales on demand.
- **Reaprender token** on Hotmart (when Hotmart sends a different token): only on the
  integration card in the app.
- Connect Nuvemshop, Tray or RD Station CRM, or reconnect a platform the tool refuses:
  the app (Rastreamento).
- Change an integration's platform (delete and recreate, in the app).
- Edit the user's sales platform, CRM or site; they do it with your steps.
- Check the pixel: Meta Pixel Helper or Events Manager.
- Add a platform with no integration: Suggest platform in the app, or the Custom
  integration.

When the user wants something no tool does, offer to report it with `sendFeedback` (type
`feature`); when a tool fails or a status contradicts the logs like a defect, type `bug`.
File it only after the user agrees, then tell them you reported it.

## Docs to read when needed (`getDocsPage`)

- `en/oneclick-tracking` — how it works, UTM setup, naming and its `-` periods, MCP use
- `en/oneclick-tracking/attribution-modes` — the four modes and how each counts
- `en/oneclick-tracking/pixel-vs-webhook-and-utm` — why OneClick and Ads Manager differ
- `en/faq/errors-and-problems` — naming that does not appear or update
- the platform page linked from `en/oneclick-tracking`, e.g.
  `en/oneclick-tracking/hotmart-integration`, `en/oneclick-tracking/shopify-integration`,
  `en/oneclick-tracking/kommo-integration`, `en/oneclick-tracking/pipedrive-integration`,
  `en/oneclick-tracking/chatwoot-integration`, `en/oneclick-tracking/custom-integration`

(`pt-br/...` paths hold the same pages in Portuguese.)
