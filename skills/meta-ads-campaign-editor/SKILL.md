---
name: meta-ads-campaign-editor
description: Safely change Meta Ads (Facebook/Instagram) campaigns, ad sets and ads that already exist through the OneClick Ads MCP — on/off, budgets and caps, bid and schedule, placements, ad set audience (locations, interests, custom audiences, Advantage+), ad copy and URL tags, creative swap, the account's monthly limit — reading the current state first, warning before changes that restart learning and confirming every write; and explain why something stopped delivering. Use when the user says "aumentar orçamento", "pausar conjunto", "ligar a campanha", "trocar público", "excluir compradores", "mudar o texto do anúncio", "colocar UTM", "mudar o lance", "por que meu anúncio parou", "quem mexeu na campanha". Requires the OneClick Ads MCP connected with write scope.
---

# Meta Ads campaign editor (OneClick Ads)

You change Meta structures that **already exist** with the OneClick Ads MCP
(`https://mcp.oneclickads.com.br/mcp`). Every write is live in the ad account and can
spend money or cost delivery: read first, show **current → new**, and get an explicit
"yes" before each write.

Talk to the user in their language (usually pt-BR) and by **names** — campaigns, ad sets,
ads, audiences, interests, places. Never ask the user for an id and never invent one; every
id comes from a listing or search tool. If the client exposes only
`search`/`describe`/`call`, search tools in English ("update ad set", "ad issues").

The docs are the source of truth for product behaviour. When a detail here is not enough,
read the page with `getDocsPage` (paths below) instead of guessing.

Not this skill: creating a campaign or adding ad sets, and adding new ads to an ad set
(`createAdsInAdSet`) are the skill `meta-ads-campaign-builder`; configuring OneClick's
automatic optimization (CPA, ROAS, maximum budget, profile) is the skill
`optimization-setup`.

## Workflow

### 1. Find the object by name

1. Ad account: `listAdAccountsFromMeta`; if there are several, ask which by name. One
   with `heldByOneClick: false` is saved first with `syncMetaAdAccountsFromMeta`.
2. Campaign: `listCampaignsFromMeta` with `nameContains` and `status: "ALL"` (the default
   returns active only, so a paused one looks missing).
3. Ad sets: `listCampaignAdSetsFromMeta`. Ads: `listCampaignAdsFromMeta` (15 per page:
   read every page before saying an ad is not there).

### 2. Read the current state

| Changing                                           | Read first                                                           |
| -------------------------------------------------- | -------------------------------------------------------------------- |
| Campaign budget, spend cap, status                 | `getCampaignFromMeta` (CBO or ABO, daily or lifetime, `spendCap`)    |
| Ad set budget, bid, schedule, placements, audience | `getAdSetFromMeta`                                                   |
| Ad copy, link, button, URL tags                    | `getCreativeCopyFromMeta` with the ad's `creativeId`                 |
| How the ad looks                                   | `getAdPreviewsFromMeta` (links valid 24 h)                           |
| Performance before deciding                        | `getCampaignLiveMetaInsights`                                        |

Show it: Meta has no undo, so the read is the record of what was.

### 3. Pick the right level and tool

- **Budget lives in one place.** CBO: `updateCampaign` `dailyBudget` (or
  `lifetimeBudget` when the campaign runs on one). ABO: `updateAdSet` on each ad set. An
  ad has no budget. Meta refuses switching a campaign between daily and lifetime.
- **Status is independent per level.** Turning a campaign on does not un-pause ad sets or
  ads paused on their own; to make something deliver, campaign, ad set and ad must all be
  ACTIVE. Pausing a parent stops everything below it.
- **Ad set**: `updateAdSet` — status, name, budgets, CBO minimum spend / daily cap, bid or
  cost cap amount, start/end time, placements.
- **Audience**: `updateAdSetAudience`, with `searchAdGeoLocationsFromMeta`,
  `searchAdInterestsFromMeta` and `listAdAccountAudiencesFromMeta` (CUSTOM rows) for ids.
- **Ad**: `updateAd` — status, name, swap to another creative of the account.
  `updateAdCopy` — primary text, headline, description, link, display link, button, URL
  tags, keeping the ad's id and history.
- **Account monthly limit**: `setMetaAdAccountMonthlyBudget` (OneClick's ceiling for the
  whole month, not a daily budget).

Field-by-field rules and what Meta refuses:
[references/change-rules.md](references/change-rules.md).

### 4. Warn about the cost of the change

- **Audience and placements** can send the ad set back into Meta's **learning phase**
  (unstable for days, may get worse). Say so, suggest testing it in a **new ad set**
  (skill `meta-ads-campaign-builder`), and apply only after they confirm — unless they
  told you not to ask.
- **Copy, link, button or creative swap**: Meta reviews the ad again; copy edits may put
  the ad set back in learning and leave likes and comments on the old post. Changing
  **only** URL tags keeps the post.
- **Bid or cost cap** too low practically stops delivery. **Turning on or raising
  budget** spends money: point out large jumps.

Per change: the table in [references/change-rules.md](references/change-rules.md).

### 5. Confirm, then write once per object

1. One summary per object: name, level, each field **current → new**, currency (values
   are in the account currency, not cents), the warning that applies. Wait for an explicit
   yes.
2. **Bundle every change to one object into one call.** Meta rate-limits writes to one
   campaign, ad set or ad: leave **30 s** between writes to the same object, counting
   every `update*` tool.
3. Send only what changes. In `updateAdSetAudience`, an omitted dimension is kept; to add
   or remove one custom audience or interest use the `add*`/`remove*` fields, never a
   rebuilt list.
4. Report the result, including what the tool says it changed on its own (placements moved
   to manual, `creativeFeaturesChanged`, `replacedOptions`, the time zone used).
5. **A failure is not always a failure.** If `updateAdSetAudience` says the change could
   not be confirmed, or any write times out or answers "Service Unavailable", do **not**
   say it failed and do not resend: read the object again, tell the user what it holds
   now, and resend only what is missing. `updateAdCopy` is not idempotent: read before
   any retry.
6. Undo of an ad copy edit: `updateAd` with the returned `previousCreativeId`.

## Why did it stop delivering?

Work top-down and say which level explains it:

1. **Status chain**: `effectiveStatus` in `listCampaignAdSetsFromMeta` /
   `listCampaignAdsFromMeta`. `CAMPAIGN_PAUSED` / `ADSET_PAUSED` means the ad is fine and
   something above it is paused; `PENDING_BILLING_INFO` is a payment matter in Meta.
2. **Rejected ad** (`DISAPPROVED`, `WITH_ISSUES`, `PENDING_REVIEW`): `getAdIssuesFromMeta`
   gives Meta's reason and the policy or placement. Empty lists mean Meta gave no detail
   through the API: send the user the `adsManagerUrl` to read the notice there.
3. **Something changed**: `listChangeHistoryFromMeta` (up to 90 days) shows edits by a
   person (`manual`) or by Meta (`meta`), with before/after and who. OneClick's own
   optimizer actions are not in it: for those, skill `optimization-setup`.
4. **Ad set settings**: `getAdSetFromMeta` — an `endTime` in the past, a low `bidAmount`,
   CBO minimums above the campaign budget (Meta refuses activation), narrowed placements.
5. **OneClick blocked**: `listMetaPendingActions` lists what only the advertiser can clear
   (Page permission, terms, payment, expired connection). Show `userError.title` and
   `userError.action` verbatim, name the blocking entity, give the `actionUrl`. A row can
   outlive the problem: confirm live before saying it blocks them now.
   - User says they fixed it → `resolveMetaPendingActions` (OneClick takes their word and
     tries again; if not fixed, it comes back).
   - User will live with it → `acknowledgeMetaPendingActions` (only the daily reminder
     stops; the block stays). Never call either on your own initiative.

## What you cannot do here

- Create campaigns, ad sets or ads (skill `meta-ads-campaign-builder`; new ads in an
  existing ad set: `createAdsInAdSet` there). Delete anything.
- Change a bid **strategy** (only the cap amount), an objective, or switch a campaign
  between daily and lifetime budget or between CBO and ABO.
- Apply a **saved** audience to an existing ad set, or edit behaviors, demographics or
  unmapped geography.
- Edit the text, link or button of a carousel, dynamic creative, catalog ad or an ad built
  from an existing post; edit the link of an ad whose button opens an Instant Form.
- Clear an ad set's location (Meta needs one; replace it).

When the user wants something no tool does, offer to report it with `sendFeedback` (type
`feature`); when a tool fails like a defect, type `bug`. File it only after the user
agrees, then tell them you reported it.

## Docs to read when needed (`getDocsPage`)

- `en/mcp/edit-campaigns-ad-sets-and-ads-with-ai` — the full editing guide
- `en/mcp/operate-campaigns-with-ai` — monthly limit, change history, pending actions
- `en/meta/ad-account-access-and-status` — access notices and pending actions
- `en/optimizations/ad-account-monthly-spend-limit` — how the monthly limit works
- `en/meta/ads-manager` — the same edits in the OneClick app

(`pt-br/...` paths hold the same pages in Portuguese.)
