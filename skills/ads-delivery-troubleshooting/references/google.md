# Google Ads — reading `listGoogleAdIssues`

Google names its reasons; read the name, quote it, then explain it. The tables cover the
common ones; for a reason not listed, read its words (`..._PAUSED`, `..._UNDER_REVIEW`,
`..._DISAPPROVED`, `..._LIMITED`) and say what it names.

## Campaign

`status` is what the user set; `servingStatus` (`SERVING`, `PENDING`, `ENDED`,
`SUSPENDED`, `NONE`) whether Google can serve it; `primaryStatus` (`ELIGIBLE`,
`LEARNING`, `LIMITED`, `NOT_ELIGIBLE`, `PENDING`, `PAUSED`, `ENDED`, `MISCONFIGURED`) Google's
overall verdict, explained by `primaryStatusReasons`.

| Reason or status                                                                        | Meaning                                                                                | Fix                                                                                                       |
| --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `BIDDING_STRATEGY_LEARNING`, `biddingStrategySystemStatus` `LEARNING_*`                 | The automatic bid strategy is learning after a start or a change                       | Wait about a week without changing bids, budget or goals. Learning alone rarely explains zero impressions |
| `BUDGET_CONSTRAINED`, `LIMITED_BY_BUDGET`                                               | The daily budget runs out                                                              | Raise the budget (`updateGoogleCampaign`) after confirming                                                |
| `LIMITED_BY_CPC_BID_CEILING`, `BIDDING_STRATEGY_CONSTRAINED`                            | A bid limit or target too strict for the auctions                                      | Loosen the target CPA/ROAS or the bid limit (`getGoogleCampaignSettings` → `updateGoogleCampaign`)        |
| `MISCONFIGURED_*` (e.g. `MISCONFIGURED_CONVERSION_TYPES`)                               | The bid strategy cannot work as set up, often a conversion goal with nothing to bid on | Check `getGoogleCampaignTargeting` `conversionGoals` and `listGoogleConversionActions`                    |
| `SEARCH_VOLUME_LIMITED`, keywords `RARELY_SERVED`                                       | Too few searches for the keywords                                                      | Broader or higher-volume keywords, broader match types (`addGoogleKeywords`)                              |
| `HAS_ADS_DISAPPROVED`, `HAS_ADS_LIMITED_BY_POLICY`, `MOST_ADS_UNDER_REVIEW`             | Ads rejected, restricted or still in review                                            | See the ads below                                                                                         |
| `AD_GROUPS_PAUSED`, `NO_AD_GROUPS`, `KEYWORDS_PAUSED`, `NO_KEYWORDS`, `NO_AD_GROUP_ADS` | Nothing enabled below the campaign                                                     | Turn on or create what is missing, after confirming                                                       |
| `servingStatus` `SUSPENDED`                                                             | Google suspended serving, usually at the account level                                 | The user resolves it in Google Ads (notices, billing, verification)                                       |
| `CAMPAIGN_ENDED`, `servingStatus` `ENDED`                                               | The end date passed                                                                    | A new end date (`updateGoogleCampaign`) after confirming                                                  |

## Ads

| Field                                                        | Meaning                                   | Fix                                                                    |
| ------------------------------------------------------------ | ----------------------------------------- | ---------------------------------------------------------------------- |
| `approvalStatus` `DISAPPROVED`                               | Rejected; `policyTopics` names the policy | Rewrite or replace the ad; an appeal is made in Google Ads             |
| `approvalStatus` `APPROVED_LIMITED`, `AREA_OF_INTEREST_ONLY` | Serves only where the policy allows       | Explain where it is limited; rewrite only if the user wants full reach |
| `reviewStatus` other than `REVIEWED`                         | Google is still reviewing                 | Wait, usually up to a business day                                     |
| `primaryStatus` `NOT_ELIGIBLE` with a reason                 | Read the reason                           | As the reason names                                                    |

## Keywords

| Field                                  | Meaning                                           | Fix                                                                      |
| -------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------ |
| `systemServingStatus` `RARELY_SERVED`  | Low search volume: Google rarely shows it         | Broader terms or match types; keep the niche ones if the user wants them |
| `approvalStatus` other than `APPROVED` | Rejected or limited by policy                     | Remove (`removeGoogleKeyword`) or replace it, after confirming           |
| `qualityScore` low (1–4)               | Ad and landing page poorly related to the keyword | Ad text closer to the keyword, a more relevant page                      |
| `qualityScore` null                    | Not enough data yet                               | Nothing to read                                                          |

## When everything looks fine and nothing serves

A campaign `ENABLED`, `SERVING`, with approved ads and eligible keywords that still has
no impressions: check whether **any** campaign of the account delivered in the last 30
days (`getGoogleAdAccountLiveInsights`). If none did, the cause is the account: payment
method missing or refused, billing pending, advertiser verification pending. No tool
reads these; send the user to **Faturamento** and the notices at the top of Google Ads.
