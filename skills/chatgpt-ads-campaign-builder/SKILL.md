---
name: chatgpt-ads-campaign-builder
description: Create and manage ChatGPT Ads (OpenAI's ad platform) campaigns, ad groups and chat-card ads through the OneClick Ads MCP — objective and the bid strategy it allows, daily or lifetime budget, locations and platforms, context hints, copy, images and URL parameters — created paused, previewed and turned on only when asked; then read performance, review status, rejections and change history, add ad groups or ads to a running campaign, and edit budget, bid, status or creative. Use whenever the user wants to advertise inside ChatGPT, e.g. "anunciar no ChatGPT", "criar campanha no ChatGPT Ads", "OpenAI Ads", "subir anúncio no ChatGPT", "por que meu anúncio do ChatGPT foi reprovado?". Requires the OneClick Ads MCP with write scope and a ChatGPT Ads account linked in OneClick.
---

# ChatGPT Ads campaign builder (OneClick Ads)

You create and manage ChatGPT Ads campaigns with the OneClick Ads MCP. ChatGPT Ads has
three levels: the **campaign** (objective and budget), the **ad group** (bid and context
hints) and the **ad** (the chat card: title, body, image, link). Every create tool makes
everything **PAUSED**; it spends only once the user asks you to turn it on. Creating and
editing still change a real account that spends money once active: show a summary and get
an explicit "yes" before every write.

Talk to the user in their language (usually pt-BR) and by **names**. Never ask for an id
and never invent one: every id (`adacct_…`, `cmpn_…`, `adgrp_…`, `ad_…`, `file_…`, location
ids) comes from a listing tool. The one exception is the conversion event of a
conversions campaign (below). With a `search`/`describe`/`call` client, search tools in
English ("create openai ads campaign").

## Workflow

### 1. Account

1. `listOpenAIAdAccounts`. Empty → no account is linked: the user links it in the OneClick
   Ads dashboard (ChatGPT Ads › Ad accounts › Link account) with an API key generated in
   the ChatGPT Ads Manager (one key per ad account). `isActive: false` → they turn it on
   there; `keyInvalid: true` → they paste a new key. Every other tool refuses those
   accounts.
2. Several accounts → ask which by name. Budgets and bids are in the account currency
   (50 = R$50, never micros).
3. To place the account in a OneClick project: `listProjects`, then
   `addOpenAIAdAccountToProject` (an account belongs to one project at most).

### 2. Read what exists

- `listOpenAIAdsCampaigns` (live, archived included): objective (`biddingType`), budget,
  targeting, `queryStringTemplate`, `servingIssues`. Check it before creating, so you do
  not duplicate a campaign: every create call creates again.
- `listOpenAIAdsAdGroups` (bid, context hints) and `listOpenAIAdsAds` (creative, review
  status, `fileId`s of images already uploaded).

### 3. Plan the campaign

Decide with the user, one value at a time:

- **Objective**: `impressions`, `clicks` or `conversions`. It sets the bid strategies the
  ad group may use and **cannot change after creation**. Table:
  [references/objectives-bids-and-limits.md](references/objectives-bids-and-limits.md).
- **Conversions** needs one active standard conversion event (`ces_…`) of the account.
  Reuse one from an existing campaign's `conversionEventSettingIds`, or ask the user which
  event to use; never invent it.
- **Budget**: `dailyBudget` **or** `lifetimeBudget`. Maximize clicks / maximize
  conversions need a daily budget. A lifetime budget can later become daily, never the
  reverse.
- **Where**: countries as ISO codes; states and cities through `findOpenAIAdsLocations`
  by name (show the canonical names, the user picks). Exclusions win over inclusions.
  Omitted: every location. **Platforms**: ChatGPT app (iOS, Android) and web; omitted:
  all.
- **Dates** (optional): `startTime`, `endTime`, ISO 8601 with offset.
- **Ad group**: name, bid (a strategy the objective allows; `maxBid` when fixed), and
  **context hints**: phrases describing the conversations where the ad is useful. Offer
  to write them from the product.
- **Ad**: internal name, **title 3–50 characters**, **body up to 100** (count before
  proposing), landing page, and one image or 4–6 square ones (multi-image only if OpenAI
  enabled it): public JPEG/PNG/WebP links ≥ 640 × 640 px, or existing `fileIds`.
- **URL parameters** (`queryStringTemplate`) per level. See
  [references/objectives-bids-and-limits.md](references/objectives-bids-and-limits.md#url-parameters).

### 4. Confirm and create

1. Show one summary: account, campaign name, objective (and conversion event), budget
   kind, amount and currency, locations and exclusions by name, platforms, dates, ad group
   name, bid strategy and max bid, context hints, each ad's title, body, link, images, the
   URL parameters per level, status PAUSED. Wait for an explicit yes.
2. Call the tool:

   | The user wants                                         | Tool                      |
   | ------------------------------------------------------ | ------------------------- |
   | A new campaign (with one ad group and one ad)          | `createOpenAIAdsCampaign` |
   | A new ad group, with its first 1–10 ads, in a campaign | `createOpenAIAdsAdGroup`  |
   | 1–10 more ads in an existing ad group                  | `createOpenAIAdsAds`      |

   A new ad group shares the campaign's budget with the others; its bid must fit the
   campaign's objective, read from `listOpenAIAdsCampaigns`.

3. The tool only **enqueues** and returns a `workerId`: poll `getWorkerStatus`. COMPLETED
   carries the ids and each ad's review status; FAILED carries OpenAI's refusal (e.g. an
   image below 640 × 640) and what was already created, paused. Fix it with the user and
   create only what is missing — check the listings first, a resend creates again.

### 5. Preview and turn on

1. `previewOpenAIAdsAd` per ad (paused ads only): a signed link, no login, valid up to 24
   hours; send it to the user, no further. Only `html` back → save it as an .html file.
2. When the user asks to turn it on, confirm, then set `status: active` on **all three**
   levels: `updateOpenAIAdsCampaign`, `updateOpenAIAdsAdGroup`, `updateOpenAIAdsAd`.
   Turning on a campaign does not turn on its paused ad groups or ads. An ad serves only
   when it, its ad group and its campaign are active and its review allows it.

### 6. Follow up

- **Performance** (live): `getOpenAIAdAccountLiveInsights` (account totals),
  `getOpenAIAdsCampaignLiveInsights` (per campaign), `getOpenAIAdsLevelLiveInsights`
  (campaign, ad group or ad). A null metric is "no data", never zero.
- **Not serving / rejected**: `listOpenAIAdsAdIssues`. Read `account` first: an inactive
  account or a brand review that did not pass holds every ad back.
- **Who changed what**: `listOpenAIAdsChangeHistory`.

Review states, rejection handling and the metric list:
[references/review-and-performance.md](references/review-and-performance.md).

## Editing

Read the current values first, show "current → new", and confirm. Each update tool changes
only the fields you send and returns what it changed.

- **Campaign** (`updateOpenAIAdsCampaign`): name, status, budget (never both kinds; the
  value replaces), description, dates, URL parameters, targeting (a field sent replaces
  it, `[]` clears it, an omitted one is kept). Budget refusals:
  [references/objectives-bids-and-limits.md](references/objectives-bids-and-limits.md#budget).
- **Ad group** (`updateOpenAIAdsAdGroup`): name, status, bid, description, context hints,
  URL parameters. Context hints **replace the whole list**: to add one, send the current
  ones plus the new. Switching to maximize clicks/conversions on a lifetime budget: make
  the campaign daily first.
- **Ad** (`updateOpenAIAdsAd`): name, status, URL parameters, title, body, link, images
  (`imageUrls` or `fileIds`, replacing all). Changing the card sends it back to review. A
  single-image ad stays single-image and a multi-image ad stays multi-image: to switch,
  create a new ad. An ad has no budget or bid.
- **Archive** (`status: archived`, sent alone): **cannot be undone**; say so and confirm.
  To stop for a while, pause instead.

## What you cannot do here

- Change a campaign's objective or conversion event, or turn a daily budget into a
  lifetime one (create a new campaign).
- Delete anything (the user deletes in the ChatGPT Ads Manager).
- Read or change the account-level default URL parameters (ChatGPT Ads Manager).
- Approve a rejected ad by turning it on: only an edited card that passes review serves.
- Preview an ad that is already active.
- Use a file from the user's computer: images need a public link or an existing `fileId`.
- Link an account: that is done in the OneClick Ads dashboard with an API key.

When the user wants something no tool does, file `sendFeedback` (type `feature`); when a
tool fails like a defect, type `bug`. Tell the user you reported it.

## Docs to read when needed (`getDocsPage`)

- `en/chatgpt-ads` — ChatGPT Ads with OneClick Ads: linking, creating, editing, targeting,
  preview, issues, URL parameters, rules
- `en/mcp` — what the OneClick Ads MCP covers

(`pt-br/chatgpt-ads` holds the same page in Portuguese.) Use `getDocsSummary` to find
other pages.
