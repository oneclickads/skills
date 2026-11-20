---
name: meta-ads-campaign-builder
description: Build a Meta Ads (Facebook/Instagram) campaign through the OneClick Ads MCP using a validated structure (Andromeda, CBO, ABO, creative testing, remarketing), the account's best creatives and the right copy, uploaded paused for review. Use whenever the user wants to create, replicate, duplicate or scale a Meta campaign, test creatives, add ad sets to a running campaign or add ads to an ad set — e.g. "criar campanha no Meta", "montar uma Escala Andrômeda", "subir campanha 1-1-10", "duplicar campanha com meus melhores anúncios", "testar criativos", "campanha de remarketing", "subir esse vídeo como anúncio". Requires the OneClick Ads MCP connected with write scope.
---

# Meta Ads campaign builder (OneClick Ads)

You build Meta campaigns with the OneClick Ads MCP (`https://mcp.oneclickads.com.br/mcp`).
Creation is real and spends money: every campaign goes up **PAUSED**, and you show the
plan and get an explicit "yes" before any write.

Talk to the user in their language (usually pt-BR) and by **names** — campaigns, ad sets,
ads, audiences. Never ask the user for an id and never invent one; every id comes from a
listing tool. If the client exposes only `search`/`describe`/`call`, search tools in
English ("copy reference campaign", "top creatives").

The docs are the source of truth for product behaviour. When a detail here is not enough,
read the page with `getDocsPage` (paths below) instead of guessing.

## Workflow

### 1. Context

1. Pick the ad account: `listAdAccountsFromMeta` (live). If the user has several, ask
   which by name.
2. Read what the account already knows: `listProjectMemories` / `searchProjectKnowledge`
   (product, sale price or average order value, URL tags, naming tags, landing page,
   Facebook Page). Reuse it instead of asking again.
3. Find candidate references: `listCampaignsFromMeta` (live; `listMetaCampaigns` only
   holds what OneClick already synced). For a reference, check its budget type (CBO or
   ABO), objective and whether it delivers.

### 2. Choose the structure

If the user named a structure ("Escala Andrômeda", "1-6-3"), use it. Otherwise call
`listValidatedCampaignStructures` and **suggest one** that fits the goal, then confirm.
How to choose the structure, size the budget and pick the creative mix is the skill
`meta-ads-campaign-structures` (installed with this one); use it for the strategy and this
skill for the build.

Quick rules if that skill is not available:

- **Goal first**: scale what already converts, or test creatives to find the next winner.
- **Andromeda family (CBO, broad Advantage+ audience)** is the default for most accounts
  today: Escala Andrômeda `1-1-10`, Teste de Criativos Andrômeda `1-1-X`,
  Escala Andrômeda II `1-X-Y`, CBO `1-1-1`. It needs creative **diversity** (formats and
  copy angles), not audience splits.
- **Fair test with equal spend per creative**: ABO `1-X-3x`.
- **Warm audiences / remarketing**: Mix Top Quente `1-4-5` (ABO, 7/30/90/180 days).
- **One proven winner to push**: Escala ABO I `1-5-1`.
- Budget from the product's sale price or average order value (Andromeda scale: 50–200%
  of it per day; Andromeda test and 1-1-1: 25–100%).

### 3. Choose the tool

| The user wants                                                                                   | Tool                                                                        |
| ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- |
| New campaign with the reference's **same objective and budget type**                             | `copyReferenceCampaign`                                                     |
| New campaign with **another budget type** (CBO↔ABO), another objective, or a chosen instant form | `createCampaign` (reference ad sets, campaign shell new)                    |
| Ad set **from scratch** (no reference: country, age, gender, custom audiences, Advantage+)       | `createCampaign`, ad sets in from-scratch mode                              |
| Add ad sets to a campaign already running                                                        | `copyReferenceCampaign` with `destination: { mode: 'EXISTING' }`            |
| Add a creative to an ad set that is already learning                                             | `createAdsInAdSet` (keeps delivery history; a cloned ad set starts at zero) |

Constraints to check before proposing:

- `copyReferenceCampaign` **cannot convert CBO↔ABO**. A CBO structure needs a CBO
  reference; otherwise use `createCampaign`, omitting `campaign.referenceCampaignId` and
  passing `objective` and `specialAdCategories` yourself.
- One ad-set mode per call: all from references or all from scratch.
- From scratch, location is **country only**. For state/city/ZIP, start from a saved
  audience (`listAdAccountAudiencesFromMeta`, `getSavedAudienceFromMeta`) or a reference.
- From scratch also needs the **bid strategy**; cost cap / bid cap need a cap value per ad
  set.
- Adding ad sets to an **active** CBO campaign starts delivery at once and redistributes
  its budget: warn the user, prefer paused ad sets.

### 4. Choose the creatives

Sources (they can be mixed): ads already in the account, an image link, a video link, a
Facebook Page post, a carousel. Details: [references/creatives-and-copy.md](references/creatives-and-copy.md).

- **Best ads**: `getTopAdAccountCreatives`. Always state the **criterion** (return,
  conversion volume or cost per conversion) and the **window** (7/30/90 days), and
  **consolidate by creative** so one winner running in several ad sets is not counted in
  pieces. Without revenue tracking, rank by conversions and cost per conversion.
- **Image/video links** must be public ("anyone with the link"), one file per creative. A
  Google Drive **folder** link cannot be passed: list its files with a Drive tool or ask
  for one link per file.
- **Video**: `uploadVideoToMediaLibrary` (confirm first), then `getVideoUploadStatus`
  until ready.
- **Image already in the account**: `searchMediaLibraryImages` by file name.
- **Page post**: `findPagePostsFromMeta` (Facebook posts only, not Instagram).

### 5. Write the copy

- Copy is **inherited field by field** from the reference ad of each ad set. Preview it
  with `getAdSetAdCopyDefaultsFromMeta` and show the **effective** copy per ad.
- **URL tags are never inherited on a new creative**: ask the user and apply them, or the
  ad goes up without tracking.
- Up to 5 primary texts, 5 headlines and 5 descriptions per new creative (not lead forms).
- The destination link is mandatory unless the ad sends to WhatsApp.
- Offer to write variations ("5 headlines in the tone of my best campaigns").

### 6. Confirm, create, follow up

1. Show one summary: account, structure and format, tool, reference, budget **per day**
   and currency, CBO/ABO, ad sets × ads, each creative, effective copy, URL tags, name,
   status PAUSED. Wait for an explicit yes.
2. Call the tool. It only **enqueues** and returns a `workerId`.
3. Poll `getWorkerStatus` until done. Campaign creation is all-or-nothing (a failure rolls
   back); `createAdsInAdSet` is best effort per ad and is not rolled back. Report every
   failed item with Meta's message. On a temporary Meta error, offer to resend.
4. CBO structures that call for minimum spend per ad set (CBO-01, CBO-04: 60% of the
   daily budget split across the ad sets) get it **after** creation with `updateAdSet`
   `dailyMinSpendTarget`, while still paused. The sum of minimums must stay below the
   campaign budget.
5. Hand off: "review it in Ads Manager, then turn it on". Offer the automatic
   optimization next (skill `optimization-setup`).

## Naming

Suggest a name that carries the structure and a tag the account already uses, e.g.
`[TAG] Escala Andrômeda 1-1-10 | Produto | 2026-11`. Tags in names let the user configure
campaigns in bulk later. Do not strip OneClick suffixes from existing names: they record
the optimizer's actions.

## Account-level best practices

So the optimizer has room to work: at least 5 active campaigns, 3+ ad sets each, many ads
per ad set, about 25% of the daily budget in tests. Andromeda structures depend on the
**reference audiences at the ad account level**; remind the user if they never set them.
Details: skill `meta-ads-campaign-structures`.

## What you cannot do here

- Attach a file from the user's computer (needs a public link).
- Run an Instagram post as an ad.
- Put several media in one ad outside a carousel.
- Create a lead form (only list and use existing ones: `listPageLeadGenFormsFromMeta`); the
  page admin must have accepted Meta's Lead Generation Terms.
- Delete campaigns.

When the user wants something no tool does, file `sendFeedback` (type `feature`); when a
tool fails like a defect, type `bug`. Tell the user you reported it.

## Docs to read when needed (`getDocsPage`)

- `en/mcp/create-campaigns-with-ai` — the full creation guide
- `en/validated-campaigns` and `en/validated-campaigns/<slug>` — each structure
- `en/meta/create-and-duplicate-campaigns` — the same flow in the app
- `en/mcp/lead-ads-forms` — instant forms
- `en/mcp/edit-campaigns-ad-sets-and-ads-with-ai` — changing what exists
- `en/optimizations/best-practices-and-measuring-performance`

(`pt-br/...` paths hold the same pages in Portuguese.)
