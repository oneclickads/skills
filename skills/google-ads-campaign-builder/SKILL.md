---
name: google-ads-campaign-builder
description: Build a Google Ads campaign through the OneClick Ads MCP — Search, Performance Max, Demand Gen, Display, Shopping or App — choosing the right type for the goal, structuring ad groups, keywords, negatives, assets and bidding, created paused for review. Use whenever the user wants to create or plan a Google Ads campaign, e.g. "criar campanha no Google Ads", "campanha de pesquisa", "subir uma Performance Max", "PMax para minha loja", "Demand Gen no YouTube", "campanha de Shopping com o Merchant Center", "anunciar meu app", "google ads mcp". Requires the OneClick Ads MCP with write scope and a Google Ads account connected in OneClick.
---

# Google Ads campaign builder (OneClick Ads)

You create Google Ads campaigns with the OneClick Ads MCP. Every create tool writes the
whole campaign at once (all or nothing) and **always creates it PAUSED**; it spends only
after the user reviews it and asks you to turn it on with `updateGoogleCampaign`. Still,
confirm every value before calling a create tool.

Speak the user's language, show everything by name, and take every id (account, location,
language, audience, image, conversion action, Merchant Center, brand list) from a listing
tool. Never type or guess an id. With a `search`/`describe`/`call` client, search in
English ("create google search campaign").

## Workflow

### 1. Account

1. `listGoogleAdAccounts`. None connected or none turned on → `createGoogleConnectLink`
   (valid 30 minutes, single use), the user authorizes in the browser and keeps "Use ad
   account" on. Then list again.
2. Several accounts → ask which by name. Note the account currency: budgets are in it,
   not in micros.
3. Read `listProjectMemories` / `searchProjectKnowledge` for product, landing page,
   brand, regions, average order value.
4. Before choosing bidding, check conversions: `listGoogleConversionActions`. No working
   conversion action → avoid conversion-based bidding and tell the user to set up
   conversion tracking first (Performance Max requires it).

### 2. Pick the campaign type

| Goal                                                         | Type            | Needs                                                   |
| ------------------------------------------------------------ | --------------- | ------------------------------------------------------- |
| Capture people already searching for the product or brand    | Search          | keywords, landing page                                  |
| Conversions at scale on every Google channel                 | Performance Max | conversion tracking, texts, images, ideally a video     |
| Create demand with visual ads (YouTube incl. Shorts, Discover, Gmail) | Demand Gen | images and/or YouTube videos; minimum daily budget  |
| Reach and remarketing with banners                           | Display         | images, logo, targeting                                 |
| Online store product ads with price and photo                | Shopping        | Merchant Center **linked** (`listGoogleMerchantCenters`) |
| App installs, engagement or pre-registration                 | App             | published app, app conversion actions                   |

Not available through the MCP: Smart and Video campaigns (use Demand Gen or Performance
Max for YouTube video), deleting campaigns, uploading videos to YouTube (the video must
already be public or unlisted on YouTube).

How to choose the type and structure ad groups, keywords, negatives, assets, bidding and
budget is the skill `google-ads-campaign-planning` (installed with this one); use it for
the plan and this skill for the build.

What the OneClick tools accept per type:

- **Search** (`createGoogleSearchCampaign`): ad groups with keywords and 1–3 responsive
  search ads each (3–15 headlines, 2–4 descriptions, optional pins), campaign negatives,
  optional Search Partners / Display Expansion (only when asked), sitelinks, callouts,
  snippets, call, images, prices, promotions, lead form.
- **Performance Max** (`createGooglePerformanceMaxCampaign`): business name and logos,
  1–10 asset groups (texts, images, YouTube videos, call to action, search themes, audience
  signal), brand exclusions, new-customer goal, Merchant Center (retail).
- **Demand Gen** (`createGoogleDemandGenCampaign`): up to 20 ad groups, up to 10 image,
  video or carousel (2–10 cards) ads each, optional audience and channels, Merchant
  Center products; minimum daily budget enforced by Google.
- **Display** (`createGoogleDisplayCampaign`): ad groups with audiences, interests, topics,
  placements, YouTube channels, contextual keywords; responsive display ads.
- **Shopping** (`createGoogleShoppingCampaign`): a LINKED Merchant Center, priority,
  product groups (all, or by brand, category, product type, item id with "everything
  else"); no language.
- **App** (`createGoogleAppCampaign`): subtype fixed at creation; installs take at most one
  download conversion action, engagement none of download, pre-registration none.

### 3. Gather the inputs

- Locations: `findGoogleGeoTargets` by name (countries, states, cities); exclusions too.
- Languages: `findGoogleLanguages` (Shopping takes none: the feed sets it).
- Audiences: `listGoogleAudiences`, `findGoogleAudienceSegments`; Display topics:
  `findGoogleTopics`.
- Images: `listGoogleImageAssets`; new ones via `uploadGoogleImageAssets` from public links.
- Brand exclusions for Performance Max: `listGoogleBrandLists`.
- Texts: write them with the user. Respect Google's limits (headline 30 characters,
  long headline 90, description 90); count before proposing.

### 4. Confirm

Show one summary: account, type, name (unique in the account), daily budget and currency,
bidding strategy and targets, networks, locations and exclusions, languages, schedule and
devices, every ad group with its keywords and match types, negatives, every text, every
image and video, assets, landing page. Wait for an explicit yes.

### 5. Create and verify

1. Call the create tool. A refusal creates nothing and names the entity, field and code
   (text too long, budget below the minimum, duplicate name, policy). Fix it with the user
   and resend.
2. Verify with `getGoogleCampaignTargeting`, `getGoogleCampaignSettings`,
   `listGoogleAdGroups`, `listGoogleAssetGroups`, `listGoogleAds`.
3. Hand off: the campaign is paused; the user reviews it in Google Ads and asks you to turn
   it on. After launch, offer keyword and negative work (`addGoogleKeywords`,
   `addGoogleNegativeKeywords`), extra assets (`addGoogleCampaignAssets`) and settings
   changes (`updateGoogleCampaign`: dates, schedule, devices, conversion goals).

## Settings every type can take (as the type allows)

Start and end dates, excluded locations, presence vs. interest in a location, ad schedule
(days and hours), device bid adjustments or excluding a device.

## Measuring sales from Google campaigns

OneClick Tracking attributes sales to Google campaigns by UTM. Put UTM parameters in the
final URL or tracking template so sales from the platforms the user sells on (Hotmart,
Kiwify, Shopify…) reach the right campaign.

## Feedback

When the user wants something no tool does, file `sendFeedback` type `feature`; when a
tool fails like a defect, type `bug`. Tell the user.

## Docs (`getDocsPage`)

- `en/mcp/create-google-ads-campaigns-with-ai` — the full guide
- `en/mcp/connect-an-ai-assistant` — connecting the MCP
- `pt-br/...` — the same pages in Portuguese
