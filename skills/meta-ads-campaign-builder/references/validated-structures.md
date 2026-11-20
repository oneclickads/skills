# Validated campaign structures (Campanhas Validadas)

The live source is the MCP tool `listValidatedCampaignStructures`: call it before
building, because the catalog can change. This page is the decision guide and a snapshot
of the catalog so you can explain the options.

## Format notation

`campaign-adsets-ads`: `1-6-3` = 1 campaign, 6 ad sets, 3 ads per ad set.
`X`/`Y` are variables the user picks; `3x` = 3 replicas of the **same** ad in the ad set.

- **CBO** (campaign budget): one budget, Meta spends it where results are best. Best for
  scaling.
- **ABO** (ad set budget): a budget per ad set. Best for fair tests with similar spend per
  variation.

## Pick in this order

1. **What is the goal?**
   - Scale → structures with goal `scale`.
   - Find new winning creatives → goal `creative-test`.
   - Both, small budget → CBO `1-1-1`.
2. **What does the account have?**
   - Many diverse creatives (videos, carousels, statics, several copy angles) → Andromeda.
   - A single proven winner → Escala ABO I `1-5-1`.
   - The 3 best ads of the last 90 days and a proven cold audience → Escala CBO I `1-6-3`.
   - A known maximum acceptable CPA and a high budget → Escala CBO II `1-6-3` with a cost
     per result goal.
   - Warm audiences big enough (pixel, engagement, checkouts) → Mix Top Quente `1-4-5`.
   - A strong interest audience and the account's 10 best ads → Escala CBO III `1-5-10`.
3. **Which budget type do the references have?** `copyReferenceCampaign` keeps it. If no
   reference has the needed type, build with `createCampaign`.
4. **Budget**: derive from the product's sale price or average order value (below), then
   confirm per day.

## Catalog snapshot

| Code    | Name (pt-BR)                 | Type | Goal          | Format | Audience                                                                                                                                                | Creatives                                                                                                          | Budget                                                                                                                                 |
| ------- | ---------------------------- | ---- | ------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| #ABO-01 | Escala ABO I                 | ABO  | scale         | 1-5-1  | Top 1 cold audience duplicated in 5 identical ad sets                                                                                                   | Top 1 creative of the last 90 days in each ad set                                                                  | Split across the 5 ad sets                                                                                                             |
| #ABO-02 | Mix Top Quente               | ABO  | scale         | 1-4-5  | Remarketing in 4 windows: 7, 30, 90, 180 days (pixel PageView + Instagram/Facebook engagement + initiated checkouts)                                    | Top 5 of the last 90 days or remarketing-specific ads; every 7–14 days duplicate in creatives validated elsewhere  | Check the engagement audience size before scaling                                                                                      |
| #ABO-05 | Teste de Criativos ABO       | ABO  | creative-test | 1-X-3x | Validated projects: Top 1 cold or broad audience. New products: warm engagement 30/90/180 days                                                          | 1 creative per ad set, 3 replicas; X = number of creatives to test                                                 | 60% of the daily budget split as ad set minimums                                                                                       |
| #CBO-01 | Escala CBO I                 | CBO  | scale         | 1-6-3  | Top 1 cold audience ×6, or broad audiences (then the creative does the targeting)                                                                       | Top 3 of the last 90 days                                                                                          | R$200–1,000/day; 60% of the daily budget as ad set minimums                                                                            |
| #CBO-02 | Escala CBO II — CPA Desejado | CBO  | scale         | 1-6-3  | Top 1 cold audience ×6, or broad                                                                                                                        | Top 2 of the last 90 days                                                                                          | Cost per result goal = maximum acceptable CPA; high daily budget; if it does not spend, create a new one with other ads and audiences |
| #CBO-03 | Escala CBO III               | CBO  | scale         | 1-5-10 | One interest audience (most obvious or most validated) ×5                                                                                               | The account's 10 best ads over the longest period, 5–10 per ad set                                                 | No ad set minimums: let Meta distribute                                                                                                |
| #CBO-04 | Teste de Criativos CBO       | CBO  | creative-test | 1-X-3x | Validated projects: Top 1 cold audience; at most 5 ad sets                                                                                              | 1 creative per ad set, 3 replicas; X ≤ 5                                                                           | 60% of the daily budget as ad set minimums                                                                                             |
| #CBO-05 | Escala Andrômeda             | CBO  | scale         | 1-1-10 | Broad, Advantage+ on, minimum age only; reference audiences set at ad account level                                                                     | 5–20 ads in different formats and copy angles (videos, carousels, statics)                                         | 50–200% of the sale price (or average order value) per day                                                                             |
| #CBO-06 | Teste de Criativos Andrômeda | CBO  | creative-test | 1-1-X  | Same as Escala Andrômeda                                                                                                                                | X = creatives to test, 5–20 recommended                                                                            | 25–100% of the sale price per day                                                                                                      |
| #CBO-07 | Escala Andrômeda II          | CBO  | scale         | 1-X-Y  | Broad, Advantage+ on, minimum age and optionally gender                                                                                                 | X ad sets = creative batches (up to 50), Y = 3–10 ads per batch; duplicate validated ad sets in, add new batches | At least 50% of the sale price per day, no ceiling                                                                                     |
| #CBO-08 | CBO 1-1-1                    | CBO  | both          | 1-1-1  | Broad, Advantage+ on, gender and minimum age                                                                                                            | 1 ad                                                                                                               | 25–100% of the sale price per day                                                                                                      |

The 90-day window can vary per project.

## Applying the budget rules

- **"60% of the daily budget as ad set minimums"** (CBO-01, CBO-04): with R$300/day and 6
  ad sets, minimums total R$180, i.e. R$30 per ad set, set after creation with
  `updateAdSet` `dailyMinSpendTarget`. For ABO-05 the same 60% becomes each ad set's own
  daily budget split.
- **Percent of the sale price**: product at R$197 → Escala Andrômeda R$99–394/day,
  Andromeda test R$49–197/day. For e-commerce use the average order value.
- Always state the result per day and in the account's currency, and point out a large
  jump compared with the reference's current budget.

## Andromeda checklist

- Broad audience, Advantage+ audience on, only minimum age (and gender for 1-1-1 / II).
- Reference audiences set in the ad account's Advantage+ audience settings
  (`https://adsmanager.facebook.com/adsmanager/manage/advertising_settings/audience_segments`).
- Creative diversity matters more than count: mix video, carousel and static, and several
  copy angles.
- Escala Andrômeda II is a living campaign: add new batches as ad sets instead of opening
  new campaigns.

## Creative-test practice

- ABO with 3 replicas of the same ad per ad set gives each creative consistent delivery.
- Up to 10 ad sets per test campaign; run batches (e.g. 10 per week).
- About 25% of the daily budget for tests, the rest in scale campaigns.
- Winners move into a scale structure (Escala Andrômeda II batches, `createAdsInAdSet` into
  a learning ad set, or a new 1-1-10).

## Profile A/B test (optional)

To find the best optimization profile for an account: create a main campaign, keep it on
the default profile, duplicate it 3 times and give each copy one of the other profiles
(see skill `optimization-setup`).

Docs: `getDocsPage` `en/validated-campaigns` and `en/validated-campaigns/<slug>`
(`abo-scale-i`, `hot-top-mix`, `abo-creative-testing`, `cbo-scale-i`,
`cbo-scale-ii-target-cpa`, `cbo-scale-iii`, `cbo-creative-testing`, `andromeda-scale`,
`andromeda-creative-testing`, `andromeda-scale-ii`, `cbo-1-1-1`).
