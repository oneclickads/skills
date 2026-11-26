---
name: ads-delivery-troubleshooting
description: Find out through the OneClick Ads MCP why a campaign, ad set, ad group or ad is not serving or serves little — on Meta (Facebook/Instagram), Google Ads, ChatGPT Ads (OpenAI) or TikTok Ads — from the account down to the ad, in the platform's own words (rejections, reviews, rarely served keywords, bid strategy learning, paused parents, disconnected accounts), then explain the cause to the user and propose the fix. Use when the user says "minha campanha não está veiculando", "não teve nenhuma impressão", "o anúncio está ativo mas não roda", "por que meu anúncio foi reprovado", "a campanha parou de entregar", "está gastando pouco", "a campanha do Google não sai do aprendizado". Requires the OneClick Ads MCP connected (write scope only for fixes).
---

# Ads delivery troubleshooting (OneClick Ads)

An ad delivers only when **every link above it** lets it: the connection to the platform,
the ad account (status, payment, verification), the campaign, the ad set or ad group, the
ad (and its review), and, on Google Search, the keywords. Find the first link that holds
it back, with read tools, from the top down, and say which level explains it.

Talk to the user in their language (usually pt-BR) and by **names**: accounts, campaigns,
ad sets, ads, keywords. Never ask the user for an id and never invent one; every id comes
from a listing tool. If the client exposes only `search`/`describe`/`call`, search tools
in English ("ad issues", "google campaigns").

Quote what the platform says (status, reason, policy) as it comes, then translate it into
plain words. Never promise impressions: an empty issue list means the platform reports
nothing blocking, not that the ad will deliver.

## 1. Which provider, which account, which campaign

1. Ask what platform if the user did not say. List the accounts by provider:
   `listMetaAdAccounts` / `listAdAccountsFromMeta`, `listGoogleAdAccounts`,
   `listOpenAIAdAccounts`, `listTikTokAdAccounts`. Several: ask which, by name.
2. Find the campaign by name: `listCampaignsFromMeta`, `listGoogleCampaigns`,
   `listOpenAIAdsCampaigns`, or `getTikTokAdsLevelLiveInsights` at the campaign level.

## 2. The connection and the account first

A broken connection or a held account stops **every** campaign, however healthy each
one looks. Check it before going down:

| Provider    | Signal                                                                                             | Meaning and fix                                                                                                       |
| ----------- | -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Meta        | `listMetaPendingActions`                                                                           | Blockers only the advertiser clears (payment, Page permission, expired connection): show them as they come            |
| Google      | `listGoogleAdAccounts`: `needsReconnect`, `isActive`, `status` (`SUSPENDED`, `CANCELED`, `CLOSED`) | Reconnect Google in the OneClick Ads app; turn the account on in the app; a suspended account is resolved with Google |
| ChatGPT Ads | `listOpenAIAdAccounts` `keyInvalid`; `listOpenAIAdsAdIssues` `account` (`status`, brand `review`)  | New API key in the app; an account or brand review that did not pass holds every ad back                              |
| TikTok      | `listTikTokAdAccounts` `grantInvalid`, `isActive`                                                  | Reconnect with `createTikTokConnectLink`                                                                              |

Then ask whether **anything** in the account delivers: the account insights for the last
30 days (`getAdAccountLiveMetaInsights`, `getGoogleAdAccountLiveInsights`,
`getOpenAIAdAccountLiveInsights`, `getTikTokAdAccountLiveInsights`). When no campaign of
the account delivers, the cause is usually the account itself: payment method missing or
refused, billing pending, advertiser verification pending. **No tool reads billing or
verification**: say so, and send the user to the platform's billing page and the notices
at the top of the account (Google Ads: **Faturamento**; Meta: **Configurações de
pagamento**).

## 3. Down the levels, by provider

### Meta

1. `effectiveStatus` in `listCampaignAdSetsFromMeta` and `listCampaignAdsFromMeta`. It
   equals `status` when all is well and diverges when the entity does not deliver:
   `CAMPAIGN_PAUSED` / `ADSET_PAUSED` (the ad is fine, a parent is paused),
   `PENDING_REVIEW`, `PENDING_BILLING_INFO` (payment, in Meta), `DISAPPROVED` /
   `WITH_ISSUES` (rejected).
2. Rejected ad: `getAdIssuesFromMeta` with the ad id gives Meta's reason and the policy or
   placement. Both lists empty: Meta gave no detail through the API; send the
   `adsManagerUrl`.
3. Something changed: `listChangeHistoryFromMeta` (edits by a person or by Meta);
   OneClick's own actions: `listOptimizationActions`.
4. Ad set settings that starve delivery: `getAdSetFromMeta` (end date in the past, a low
   bid or cost cap).

Fixing it (status, budget, copy, resolving a pending action): skill
`meta-ads-campaign-editor` → "Why did it stop delivering?".

### Google Ads

1. `listGoogleAdIssues` with the `customerId` and the `campaignId` (omit it for every
   enabled campaign). It returns Google's own diagnosis:
   - each campaign's `primaryStatus` and `primaryStatusReasons`, `servingStatus` and
     `biddingStrategySystemStatus`;
   - the enabled ad groups, ads and keywords Google holds back: a `primaryStatus` other
     than `ELIGIBLE`, an ad not `APPROVED` or still in review (with `policyTopics`), a
     keyword `RARELY_SERVED` (low search volume) or not approved, with its `qualityScore`.
     Worst first; at most 100 per level, the rest counted in `omitted`.
2. Read the campaign first, then the children. What each reason means and the fix:
   [references/google.md](references/google.md).
3. Budget and bids behind a `LIMITED` campaign: `getGoogleCampaignSettings` (daily budget,
   bid strategy, target CPA or ROAS). Conversion goals the strategy bids on:
   `getGoogleCampaignTargeting` `conversionGoals`, and `listGoogleConversionActions` for
   whether they record anything.
4. Volume over time: `getGoogleAdsLevelLiveInsights` (campaign, ad group or ad, every
   preset period in one call).

Performance Max asset groups are not diagnosed by `listGoogleAdIssues`:
`listGoogleAssetGroups` shows only whether each is on or off; Google's diagnosis of an
asset group is read in Google Ads.

### ChatGPT Ads (OpenAI)

`listOpenAIAdsAdIssues`: read `account` first, then each ad with its `reviewStatus`,
`review` and `servingIssues`, and its ad group and campaign. Rejected ad, review states
and fixes: skill `chatgpt-ads-campaign-builder` →
`references/review-and-performance.md`.

### TikTok Ads

`getTikTokAdsLevelLiveInsights` at the campaign, ad group and ad levels: each row's
`status` and `deliveryStatus` (TikTok's `secondary_status`) with its windows; entities
with no delivery are listed too. Quote `deliveryStatus` as it comes; the first level whose
`deliveryStatus` is not delivering explains the rest. Status changes:
`updateTikTokCampaign`, `updateTikTokAdGroup`, `updateTikTokAd`, after confirming.

## 4. Explain and propose

Answer in this order:

1. **The cause**, at the level that explains it, most likely first, in plain words with
   the platform's term in parentheses ("the keywords have too little search volume —
   `RARELY_SERVED`").
2. **What is fine**, briefly, so the user does not chase it.
3. **The fix**: what the user does in the platform (billing, verification, an appeal) and
   what the assistant can do through the MCP.

Every fix writes on a live account and may spend money: state the change in plain words
(current → new) and get an explicit yes before calling any write tool. Never turn
something on, raise a budget or add keywords on your own initiative.
