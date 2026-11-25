# Review, serving issues, performance and change history

## Review status

`listOpenAIAdsAds` returns each ad's `reviewStatus`: `in_review`, `rejected` or
`approved`, plus `review` (OpenAI's detail, as it comes) and `servingIssues`.

| Situation                                                      | What to do                                                                                                                                                                                                  |
| -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| New ad                                                         | `getWorkerStatus` reports each ad's review status at creation.                                                                                                                                              |
| `in_review`                                                    | Wait; it serves once approved and every level is active.                                                                                                                                                    |
| `rejected`                                                     | Read the reason in `listOpenAIAdsAdIssues` (`ad.review`). Propose a new title, body, link or images, confirm, then `updateOpenAIAdsAd`; it goes back to `in_review`. Turning it on does **not** approve it. |
| Rejected multi-image ad, user wants one image (or the reverse) | Create a new ad with `createOpenAIAdsAds`; an ad keeps its kind.                                                                                                                                            |
| Editing the card of an approved ad                             | Warn that it goes back to review and stops serving until approved again.                                                                                                                                    |

## Why an ad is not serving — `listOpenAIAdsAdIssues`

Returns:

- `account`: `status`, `review` (the advertiser brand review), `accountIntegrityReview`
  (null when OpenAI returns none). **Read it first**: an account that is not active or
  whose review did not pass holds every ad back, listed or not.
- `ads`: each non-archived ad with serving issues of its own, a `reviewStatus` of
  `rejected` or `in_review`, or that is active under a campaign or ad group that is not
  active or has serving issues. Each entry carries the ad, its `adGroup` and its
  `campaign` (id, name, status, `servingIssues`).

Without `campaignId` it reads every non-archived campaign; with it, only that one
(archived or not). An empty `ads` list does not promise impressions. `servingIssues` are
passed as OpenAI returns them: quote them, do not reinterpret them.

Common causes and the fix, by level:

| Cause                                  | Fix                                                                                     |
| -------------------------------------- | --------------------------------------------------------------------------------------- |
| Campaign or ad group paused            | `updateOpenAIAdsCampaign` / `updateOpenAIAdsAdGroup` `status: active`, after confirming |
| Ad rejected                            | Edit the card (above)                                                                   |
| Ad in review                           | Wait                                                                                    |
| Account or brand review pending/failed | Nothing here: the user resolves it with OpenAI                                          |
| Archived                               | Cannot be turned on again; create a new one                                             |

## Performance

All live from OpenAI, money in the account currency.

| Tool                               | Gives                                                                                                                                                     |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `getOpenAIAdAccountLiveInsights`   | account totals for one `datePreset`                                                                                                                       |
| `getOpenAIAdsCampaignLiveInsights` | every campaign (or one `campaignId`), all seven periods unless `datePresets` narrows them                                                                 |
| `getOpenAIAdsLevelLiveInsights`    | rows per `campaign`, `adGroup` or `ad`, optionally inside one campaign, ad group or ad; custom `clickAttributionWindowDays` / `viewAttributionWindowDays` |

- Periods: `today`, `yesterday`, `last_7d`, `last_14d`, `last_30d`, `this_month`,
  `last_month`, in the account's time zone. `last_Nd` ends yesterday; `today` and
  `this_month` include today.
- Metrics: spend, impressions, clicks, conversions (click-through plus view-through, as
  OpenAI attributes them; default 30-day click, 1-day view), conversionsValue, ctr,
  averageCpc, cpm, costPerConversion, roas.
- A metric OpenAI did not return, or a ratio with a zero divisor, is `null` — say "no
  data", never 0. The newest days are provisional as of `fetchedAt`.
- A campaign or entity with no activity since the first day of last month may be absent.
- Conversions are OpenAI's own attribution.

## Change history — `listOpenAIAdsChangeHistory`

- Every change to campaigns, ad groups, ads and other resources, by whoever made it
  (OneClick included), newest first (`order: asc` for oldest first).
- Filters combine: `campaignId` (includes its ad groups and ads), `adGroupId` (includes its
  ads), `adId`, `actorId` (from a previous item), `since`, `until` (a bare date is its
  first instant in UTC: pass the next day to include a whole day).
- `limit` 1–100 (default 50). Page with `nextCursor` → `cursor`, sending the same filters
  and order; the history ends when `nextCursor` is absent.
- Each item: `type` (e.g. `update_campaign`), `objectType`, `objectId`, `objectName`,
  `actorType`, `actorId`, `eventTime` (may be null), and `changes` / `hierarchyInfo`
  (OpenAI's raw detail; shape depends on the type, e.g. a name's before and after).
  Summarize it by name for the user.
