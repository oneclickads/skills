---
name: oneclick-ads-onboarding
description: Guide a OneClick Ads user through account setup with the OneClick Ads MCP — connect Meta and enable an ad account, turn on OneClick Tracking for their sales platform, create the first optimization — and finish with a first diagnosis of the account. Use when the user says "start onboarding", "iniciar onboarding na OneClick", "configurar minha conta", "o que falta configurar?", "continuar meu onboarding", "acabei de assinar a OneClick", or connects the OneClick Ads MCP for the first time.
---

# OneClick Ads onboarding

Walk the user from a fresh account to a running optimization, one step at a time,
checking each step with the MCP before moving on. Progress lives in the account, not in
the conversation: the user can stop and resume days later in any assistant.

Speak the user's language, by names, never ask for ids.

## 0. Preconditions

- The MCP is connected (`https://mcp.oneclickads.com.br/mcp`). For setup steps that write
  (connect links, tracking, optimization) it needs **write** scope; if a write fails with a
  scope error, tell the user to reconnect with read and write.
- An **active subscription** unlocks everything. Without one, only the billing tools work:
  `getSubscription`, `listSubscriptionPlans`, `createCheckoutLink`,
  `getPaymentUpdateLink`. Offer the plans and the checkout link, then continue after
  payment (`syncSubscription` if the status looks stale).

## 1. Read the checklist

Call `getOnboardingStatus` first, and again after each step. It lists the setup steps and
which are done (detected automatically). Tell the user where they are in one or two lines
and start at the first pending step.

## 2. Connect Meta and enable an ad account

1. `createMetaConnectLink` → give the user the link (single use, 30 minutes). Never open
   it yourself or share it.
2. After they authorize, `listMetaAdAccounts` (or `syncMetaAdAccountsFromMeta` if the list
   looks stale) to confirm the accounts arrived.
3. The user picks which ad account(s) to use; enable it in OneClick with
   `updateMetaAdAccount` (`isActive: true`) after confirming. The plan may limit how many
   active accounts they can have.
4. If an account shows access problems, explain from `getMetaAdAccount` and the docs page
   `en/meta/ad-account-access-and-status`; a reconnect is a new connect link.

Google Ads is optional here: `createGoogleConnectLink`, then `listGoogleAdAccounts`.

## 3. Turn on OneClick Tracking

OneClick Tracking receives sales from the platform's webhook and attributes them to
campaigns by UTM.

1. Ask where they sell (Hotmart, Kiwify, Eduzz, Shopify, Yampi, a CRM such as Pipedrive or
   Kommo…). Check what already exists: `listTrackingIntegrations`.
2. Read the platform's docs page for the specifics:
   `getDocsPage` `en/oneclick-tracking/<platform>-integration` (index:
   `en/oneclick-tracking`).
3. Confirm platform, currency, attribution mode and campaign-name indicators, then
   `createTrackingIntegration`. Nuvemshop, Tray and RD Station CRM connect only in the app
   (Rastreamento).
4. Give back exactly what the tool returns, each in its own code block, never retyped: the
   `webhookUrl` to paste in the platform, the `utmParameters` for the ads, and the
   `utmPassThroughScript` for the sales pages.
5. Verify sales arrive: `countMetaTrackingRawLogs` / `getMetaTrackingRawLogs` after a test
   or real sale; `getMetaAdAccountTrackingInsights` once there is volume.

## 4. First optimization

1. `listCampaignsFromMeta` → help the user pick an active conversion campaign by name.
2. `listAvailableAlgorithms` for it. No algorithms means the objective is not supported
   yet: say so and offer `sendFeedback` (type `feature`).
3. Agree the parameters (profile, sale price, cost per conversion to pause or minimum
   ROAS, maximum daily budget). Use the skill `optimization-setup` for how to derive each
   value.
4. Confirm and `createOptimization`.

## 5. First diagnosis

Close with a short read of the account so the user sees value right away:

- `getAdAccountLiveMetaInsights` for the last 30 days: spend, conversions, cost per
  conversion, ROAS if tracked.
- `getTopAdAccountCreatives` (last 30–90 days, consolidated by creative): the 3–5 best ads.
- `listCampaignsWithOptimizationStatus`: active campaigns not yet under OneClick.
- Against the OneClick best practices (at least 5 active campaigns, 3+ ad sets each, many
  ads per ad set), point out the gap and suggest the next move, e.g. a validated structure
  built from the best ads (skill `meta-ads-campaign-builder`).

## 6. Rewards

Each completed step earns points on the Achievements page. `listGamificationRewards`
shows the levels reached; `requestGamificationReward` requests one (returns the support
WhatsApp link). Celebrate briefly when a step completes.

## Docs (`getDocsPage`)

- `en/mcp/ai-guided-onboarding`
- `en/mcp/connect-an-ai-assistant`
- `en/meta/link-a-new-ad-account`
- `en/oneclick-tracking` and the platform pages
- `en/optimizations/configure-campaigns`
