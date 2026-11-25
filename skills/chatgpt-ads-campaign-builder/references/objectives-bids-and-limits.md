# Objectives, bids, budgets and field limits

## Objective → billing → bid strategies

The campaign's objective is fixed at creation. The ad group's bid strategy must match it.

| Objective     | Billing (tool contract) | Allowed bid strategies              | `maxBid` means                                |
| ------------- | ----------------------- | ----------------------------------- | --------------------------------------------- |
| `impressions` | per impression          | `fixed_bid`                         | max bid per impression (a R$60 CPM is `0.06`) |
| `clicks`      | per click               | `fixed_bid`, `maximize_clicks`      | max bid per click                             |
| `conversions` | per click (see note)    | `fixed_bid`, `maximize_conversions` | the CPA (target cost per conversion)          |

- `maximize_clicks` and `maximize_conversions` need the campaign on a **daily** budget,
  bill per click and clear audience bid multipliers. `fixed_bid` keeps the ad group's
  billing event and audience bid multipliers.
- `conversions` needs `conversionEventSettingId`: one active standard conversion event
  (`ces_…`) of the account; refused for the other objectives. Fixed after creation.
- Note: the docs' FAQ says you pay "per impression, per click or per conversion,
  depending on the objective", while the tool contract says a conversions campaign bills
  per click. Do not promise either: read the ad group's `biddingConfig.billingEventType`
  (`impression` or `click`) from `listOpenAIAdsAdGroups` and report what it says.

## Budget

| Rule                                                            | Source                          |
| --------------------------------------------------------------- | ------------------------------- |
| `dailyBudget` or `lifetimeBudget` (total for the campaign)      | create / update tools           |
| Amounts in the account currency, not micros (50 = R$50)         | every money field               |
| Never both in one update; the new value replaces the old        | `updateOpenAIAdsCampaign`       |
| Lifetime → daily allowed; daily → lifetime needs a new campaign | `updateOpenAIAdsCampaign`       |
| Refused: below the currency's minimum                           | `updateOpenAIAdsCampaign`, docs |
| Refused: lifetime budget below what the campaign already spent  | `updateOpenAIAdsCampaign`, docs |
| Refused: daily budget below an ad group's effective bid         | `updateOpenAIAdsCampaign`, docs |
| A new ad group shares the campaign's budget                     | `createOpenAIAdsAdGroup`        |

The currency minimum is OpenAI's and no tool states it: when OpenAI refuses, relay its
message and adjust with the user.

## Field limits

| Field                                                        | Limit                                                                                                                |
| ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| Campaign / ad group / ad name                                | 3–1000 characters on create (ad name is internal, never shown)                                                       |
| Ad title                                                     | 3–50 characters, shown on the card                                                                                   |
| Ad body                                                      | 1–100 characters, shown on the card                                                                                  |
| `targetUrl`                                                  | a URI, where a click lands (one per ad, for every image)                                                             |
| Images                                                       | JPEG, PNG or WebP, ≥ 640 × 640 px, public URL OpenAI downloads                                                       |
| Images per ad                                                | 1, or 4–6 distinct square images in display order (multi-image needs an account OpenAI enabled)                      |
| `imageUrls` vs `fileIds`                                     | one or the other, never both; `fileIds` come from `listOpenAIAdsAds` (`creative.fileId`, `creative.images[].fileId`) |
| Ads per `createOpenAIAdsAds` / `createOpenAIAdsAdGroup` call | 1–10, created in order                                                                                               |
| Context hints                                                | up to 2000 free-form phrases or keywords                                                                             |
| `locationIds` / `excludedLocationIds`                        | up to 2500 each, from `findOpenAIAdsLocations`                                                                       |
| `countries` / `excludedCountries`                            | two-letter ISO codes                                                                                                 |
| `platforms`                                                  | `ios_app`, `android_app`, `web` (every browser), `ios_web`, `android_web`, `desktop_web`                             |
| `startTime` / `endTime`                                      | ISO 8601 with offset; `null` clears on update                                                                        |
| `findOpenAIAdsLocations` `limit`                             | 1–50, default 10                                                                                                     |

## URL parameters

`queryStringTemplate` exists on the campaign, the ad group and the ad.

- Format: `key=value` pairs joined by `&`, no leading `?`. Example from the tool contract:
  `utm_source=chatgpt-oneclick-ads&utm_medium={ad_group_id}&utm_campaign={campaign_id}&utm_content={ad_id}`.
- Click-time macros: `{campaign_id}`, `{ad_group_id}`, `{ad_id}`, `{oppref}`.
- Hierarchy: account < campaign < ad group < ad. Every level's template is appended to
  the landing page; on the same key the most specific level wins.
- Sending a template **replaces that level's whole template**: read the current one from
  the listing first and keep the parameters already in use.
- The account-level default lives in the ChatGPT Ads Manager and no tool reads it. A
  `null` template on a campaign does not mean untagged clicks.
- OneClick's own convention (docs): `utm_source=openai`, the campaign in `utm_campaign`,
  the ad in `utm_content`. The OneClick dashboard's **Apply UTMs** button (ChatGPT Ads ›
  Ad accounts) writes these on the account's campaigns, keeping other parameters. Ask the
  user whether they already tag, and do not overwrite a convention they rely on.
