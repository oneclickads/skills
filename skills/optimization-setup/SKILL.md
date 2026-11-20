---
name: optimization-setup
description: Configure and turn on OneClick Ads automatic optimization for Meta campaigns through the OneClick Ads MCP — choose the optimization profile, derive the sale price, cost per conversion to pause or minimum ROAS and maximum daily budget from the account's history, apply it to one campaign or many, and set the account's monthly spend limit. Use after creating a campaign, or when the user says "otimizar campanha", "ligar a OneClick nessa campanha", "configurar CPA/ROAS", "qual CPA colocar?", "pausa rigorosa", "escala acelerada", "limite mensal da conta", "configurar todas as campanhas com a tag X".
---

# OneClick Ads optimization setup

The OneClick engine pauses, scales and protects Meta campaigns from parameters the user
sets. Those parameters are the "prompt" the engine follows, so derive them from the
account's data, explain the reasoning, and confirm before writing. Every write changes
live campaigns.

## Workflow

1. **Find the campaigns**: `listCampaignsFromMeta` by name, or
   `listCampaignsWithOptimizationStatus` to see which are already under OneClick.
2. **Algorithms**: `listAvailableAlgorithms` per campaign (sales vs. leads). No algorithm
   means the objective is not supported yet: say so, offer `sendFeedback` type `feature`.
3. **Account standard**: `listOptimizations` / `getOptimization` on the account's active
   optimizations. Reusing the values already in use keeps the account consistent; ask
   whether to follow it.
4. **Derive the values** (below) from `getAdAccountLiveMetaInsights`,
   `getCampaignLiveMetaInsights` or `getCampaignContextInsights` over the last 30 days.
   Present a conservative and an aggressive option, with the reasoning.
5. **Validate against what converts**: before applying a cutoff to running campaigns,
   check whether the best ads' cost per conversion (or ROAS) sits near the cutoff. If it
   would pause the ad that carries the conversions, suggest more headroom.
6. **Confirm and write**: `createOptimization` for one campaign, `createOptimizationBulk`
   for several; `updateOptimization` / `updateOptimizationBulk` to change. Bulk tools
   report per item: resend only what failed or was not attempted.

## The parameters

### Profile (Pause × Scaling)

| Profile        | Pause    | Scaling      | Behaviour                                                                 |
| -------------- | -------- | ------------ | ------------------------------------------------------------------------- |
| Default        | Flexible | Conservative | Waits longer before pausing; gradual scaling. Start here.                 |
| 1              | Strict   | Accelerated  | Pauses as soon as the CPA is hit; frequent scaling, no hourly scaling.    |
| 2              | Strict   | Conservative | Fast protection, slow scaling.                                            |
| 3              | Flexible | Accelerated  | Patient pauses plus intraday (hourly) scaling that resets next day.       |

- Changing a running campaign's profile first **undoes** the pauses and budget cuts of
  the old profile: what it paused becomes active again. Warn before changing.
- Not sure which fits? Profile A/B test: one main campaign on the default, three copies
  each on another profile, compare.
- The account default profile (`updateMetaAdAccountSettings`) applies to campaigns
  optimized from then on, not to existing ones.

### Sale price (sales campaigns only)

Digital product: its exact price. E-commerce or several kits: the actual **average order
value** from the store. Lead campaigns have no sale price.

### Cost per conversion to pause (stop loss)

- **Derivation**: the account's or campaign type's average CPA over the last 30 days
  **+ 25–30%**. Average R$100 → R$125.
- **How the engine reads it**: up to ~80% of the value it **scales**, 80–100% it
  **maintains**, above 100% it **pauses**. During an ad set's first 7 conversions it
  tolerates up to ~1.6× before pausing (learning phase).
- **Guards**: never below 1/4 of the sale price (premature pauses), never above the sale
  price (no return possible) — unless the user knowingly accepts a higher CPA on a cheap
  product (e.g. price R$19, acceptable CPA R$55: enter both as they are).
- In tests, a slightly higher value buys sample; lower it later.

### Minimum ROAS (instead of CPA)

Sales campaigns only; mutually exclusive with CPA. Use for e-commerce where return is the
metric. Suggest from the last 90 days' average return, e.g. the target the business needs
(3.5 → structures below 3.5 pause). Minimum accepted value 0.1.

### Maximum daily budget (scaling ceiling)

- Not the starting budget (that stays in Meta): the ceiling the engine may scale up to.
  Per ad set on ABO, per campaign on CBO. Minimum R$7.
- Treat it as a safety lock. Confirm it is **per day**; point out jumps of 10× or more.
- **Unlimited scaling**: no per-campaign ceiling; use only with an account monthly limit
  as protection.

### Account monthly spend limit

`setMetaAdAccountMonthlyBudget`: one ceiling for the whole account (R$30,000/month ≈
R$1,000/day). With it set, the per-campaign maximum becomes optional and unlimited scaling
is safe. Without it, each campaign needs a maximum daily budget or an explicit unlimited
choice. Budget freed by paused campaigns flows to the ones performing.

## Bulk configuration

Describe the criterion and apply in bulk: a tag in the campaign name, CBO vs. ABO, active
only, with exceptions ("all sales campaigns except those with CALENDLY"). List the matched
campaigns by name and confirm the set before writing.

## After setup

- Let the engine work: avoid manual interference; the engine pauses, scales and duplicates
  on a plan.
- Suffixes in names record its actions (temporary pauses reactivate the next day); read
  them with the docs page `en/optimizations/suffix-system`. Suffixes can be turned off per
  account (`updateMetaAdAccountSettings`).
- Follow decisions with `getOptimizationState`, `getDailyIntelligenceReport` and
  `listChangeHistoryFromMeta`.
- Measure with a sample: at least 5 active campaigns, 3+ ad sets each; compare periods
  with and without OneClick, and the account's average CPA over time.

`deactivateAllOptimizations` and turning optimizations off stop the protection: only on an
explicit request, after confirming the scope.

## Docs (`getDocsPage`)

- `en/optimizations/configure-campaigns`
- `en/mcp/operate-campaigns-with-ai`
- `en/optimizations/ad-account-monthly-spend-limit`
- `en/optimizations/pauses-and-protection`
- `en/optimizations/suffix-system`
- `en/optimizations/best-practices-and-measuring-performance`
