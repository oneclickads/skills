# Google Ads — reading `listGoogleAdIssues`

Each issue's `code` is Google's own name; read it, quote it, then explain it. The tables
cover the common ones; for a code not listed, read its words (`..._PAUSED`,
`..._UNDER_REVIEW`, `..._DISAPPROVED`, `..._LIMITED`) and say what it names.

## Campaign

`status` is what the user set. A campaign's issues carry three kinds of code: its
`servingStatus` when not `SERVING` (`PENDING`, `ENDED`, `SUSPENDED`, `NONE`: whether Google
can serve it), each primary status reason (Google's verdict; with no reason, the primary
status itself: `LEARNING`, `LIMITED`, `NOT_ELIGIBLE`, `PENDING`, `MISCONFIGURED`…), and its
bid strategy status when not `ENABLED` (message "Bid strategy: …").

| Code                                                                                    | Meaning                                                                                | Fix                                                                                                       |
| --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `BIDDING_STRATEGY_LEARNING`, bid strategy `LEARNING_*`                                  | The automatic bid strategy is learning after a start or a change                       | Wait about a week without changing bids, budget or goals. Learning alone rarely explains zero impressions |
| `BUDGET_CONSTRAINED`, `LIMITED_BY_BUDGET`                                               | The daily budget runs out                                                              | Raise the budget (`updateGoogleCampaign`) after confirming                                                |
| `LIMITED_BY_CPC_BID_CEILING`, `BIDDING_STRATEGY_CONSTRAINED`                            | A bid limit or target too strict for the auctions                                      | Loosen the target CPA/ROAS or the bid limit (`getGoogleCampaignSettings` → `updateGoogleCampaign`)        |
| `MISCONFIGURED_*` (e.g. `MISCONFIGURED_CONVERSION_TYPES`)                               | The bid strategy cannot work as set up, often a conversion goal with nothing to bid on | Check `getGoogleCampaignTargeting` `conversionGoals` and `listGoogleConversionActions`                    |
| `SEARCH_VOLUME_LIMITED`, keywords `RARELY_SERVED`                                       | Too few searches for the keywords                                                      | Broader or higher-volume keywords, broader match types (`addGoogleKeywords`)                              |
| `HAS_ADS_DISAPPROVED`, `HAS_ADS_LIMITED_BY_POLICY`, `MOST_ADS_UNDER_REVIEW`             | Ads rejected, restricted or still in review                                            | See the ads below                                                                                         |
| `AD_GROUPS_PAUSED`, `NO_AD_GROUPS`, `KEYWORDS_PAUSED`, `NO_KEYWORDS`, `NO_AD_GROUP_ADS` | Nothing enabled below the campaign                                                     | Turn on or create what is missing, after confirming                                                       |
| `SUSPENDED` (serving status)                                                            | Google suspended serving, usually at the account level                                 | The user resolves it in Google Ads (notices, billing, verification)                                       |
| `CAMPAIGN_ENDED`, `ENDED` (serving status)                                              | The end date passed                                                                    | A new end date (`updateGoogleCampaign`) after confirming                                                  |

## Ads

An ad's issues: its approval when not `APPROVED` (`kind` `policy`, `detail.policyTopics`
naming the policy), its review when not `REVIEWED` (`kind` `review`), then any other
primary status reason.

| Code                                        | Meaning                                   | Fix                                                                    |
| ------------------------------------------- | ----------------------------------------- | ---------------------------------------------------------------------- |
| `DISAPPROVED`                               | Rejected; `policyTopics` names the policy | Rewrite or replace the ad; an appeal is made in Google Ads             |
| `APPROVED_LIMITED`, `AREA_OF_INTEREST_ONLY` | Serves only where the policy allows       | Explain where it is limited; rewrite only if the user wants full reach |
| `REVIEW_IN_PROGRESS`, `UNDER_APPEAL`…       | Google is still reviewing                 | Wait, usually up to a business day                                     |
| Another reason (`kind` `serving`)           | Read the reason                           | As the reason names                                                    |

## Keywords

Keywords are not a level of their own: each keyword Google holds back is an issue of its
**ad group**, with the keyword in `detail` (`text`, `matchType`, `status`,
`systemServingStatus`, `approvalStatus`, `qualityScore`) and its words in `message`
(`Keyword "tenis" (PHRASE): rarely served (low search volume)`).

| Code or detail                       | Meaning                                           | Fix                                                                      |
| ------------------------------------ | ------------------------------------------------- | ------------------------------------------------------------------------ |
| `RARELY_SERVED`                      | Low search volume: Google rarely shows it         | Broader terms or match types; keep the niche ones if the user wants them |
| `KEYWORD_DISAPPROVED`, `KEYWORD_<…>` | Rejected or limited by policy                     | Remove (`removeGoogleKeyword`) or replace it, after confirming           |
| `detail.qualityScore` low (1–4)      | Ad and landing page poorly related to the keyword | Ad text closer to the keyword, a more relevant page                      |
| `detail.qualityScore` null           | Not enough data yet                               | Nothing to read                                                          |

## When everything looks fine and nothing serves

A campaign `ENABLED` with no issue listed, nor any for its ad groups and ads, that still
has no impressions: check whether **any** campaign of the account delivered in the last 30
days (`getGoogleAdAccountLiveInsights`). If none did, the cause is the account: payment
method missing or refused, billing pending, advertiser verification pending. No tool
reads these; send the user to **Faturamento** and the notices at the top of Google Ads.
