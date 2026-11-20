# Google Ads campaign types — structure and best practices

"MCP" lines state what the OneClick Ads tools accept. "Practice" lines are standard Google
Ads practice to propose to the user; confirm before applying.

## Search — `createGoogleSearchCampaign`

**MCP**: ad groups, each with keywords (exact, phrase or broad) and 1 to 3 responsive
search ads (3–15 headlines, 2–4 descriptions); headlines and descriptions can be pinned;
campaign negative keywords; landing page. Optional: Search Partners or Display Expansion
(only when asked), sitelinks, callouts, structured snippets, call, images, prices,
promotions, lead form. Bidding: maximize clicks, maximize conversions (optional target
CPA), maximize conversion value (optional target ROAS), manual CPC, target impression
share.

**Practice**:

- Separate **brand** and **non-brand** into different campaigns: brand is cheap and
  protects the name; non-brand needs its own budget and targets.
- One theme per ad group (one intent, closely related keywords) so ads and landing page
  match the search.
- Start with exact and phrase match; add broad match when conversion bidding has data.
- Ship negatives from day one: "grátis", "gratuito", "download", "pdf", "vagas",
  "emprego", "curso" (unless they sell courses), "reclame aqui", competitor terms the user
  does not want.
- Fill the RSA: aim for 10–15 headlines and 4 descriptions, with the keyword in several
  headlines; pin the brand to position 1 only when the user asks (pinning reduces
  combinations).
- Add at least 4 sitelinks and 4 callouts.
- No conversion history: start on maximize clicks (optionally with a CPC cap via manual
  CPC) and move to maximize conversions once the account records conversions steadily.
- Keep Search Partners and Display Expansion off unless the user wants more reach.

## Performance Max — `createGooglePerformanceMaxCampaign`

**MCP**: business name and logos, 1 to 10 asset groups (headlines, long headlines,
descriptions, landscape and square images, optional portrait, YouTube videos, landing page,
call to action, search themes, an audience signal), optional sitelinks, callouts,
snippets, call, brand exclusions, new-customer goal, and Merchant Center products (retail
PMax). Bidding: maximize conversions (optional target CPA) or maximize conversion value
(optional target ROAS). Conversion tracking is required.

**Practice**:

- One asset group per product line or audience theme, each with its own landing page.
- Fill every asset slot Google allows; include at least one YouTube video, or Google
  generates a lower-quality one.
- Use search themes and the customer list / site visitor audiences as signals.
- Exclude the user's own brand terms with a brand list when a brand Search campaign exists,
  so PMax does not take credit for brand searches.
- E-commerce with Merchant Center: retail PMax or Shopping; tROAS once there is revenue
  data.

## Demand Gen — `createGoogleDemandGenCampaign`

**MCP**: up to 20 ad groups, each with location, language and up to 10 ads: image,
YouTube video, or carousel (2–10 cards). Optional saved audience per group (without one,
Google finds the audience), channel selection (all by default), Merchant Center products.
Bidding: maximize conversions, maximize conversion value or maximize clicks. Google
enforces a minimum daily budget; below it nothing is created and the error says the
minimum in the account currency.

**Practice**:

- Test audience vs. no audience as separate ad groups (e.g. one with a saved audience, one
  optimized by Google).
- Supply vertical video for Shorts and both landscape and square images.
- It is a prospecting channel: judge it on assisted results and cost per conversion over
  weeks, not days.

## Display — `createGoogleDisplayCampaign`

**MCP**: Display Network only; ad groups with targeting (audiences, interests, topics,
sites, YouTube channels, contextual keywords) and responsive display ads (headlines, long
headline, descriptions, business name, images, logo). Bidding: manual CPC, manual viewable
CPM, maximize clicks, maximize conversions or maximize conversion value.

**Practice**: best for remarketing site visitors and for cheap reach; exclude mobile apps
placements if the user sees junk traffic; frequency matters more than reach for
remarketing.

## Shopping — `createGoogleShoppingCampaign`

**MCP**: needs a Merchant Center **linked** to the account (`listGoogleMerchantCenters`,
LINKED only). Campaign priority, ad groups with a product ad and product groups: all
products, or split by brand, category, product type or item id with an "everything else"
group. Bidding: manual CPC (bid per product group), maximize clicks, target ROAS or
maximize conversion value. No language field.

**Practice**: split product groups by margin or best sellers so they get separate bids;
start with all products and split once data shows winners.

## App — `createGoogleAppCampaign`

**MCP**: subtype installs, engagement (already installed) or pre-registration (Google
Play only, not launched yet), which cannot change later. Needs the app published and app
conversion actions (`listGoogleConversionActions`): installs take at most one download
action; engagement takes no download action; pre-registration takes none. One ad group and
one app ad with texts, images and videos that Google combines.

## Budget sanity

- Daily budget is an average; Google can spend up to 2× on a given day while keeping the
  monthly average.
- Tie the budget to targets: with a target CPA of R$50, a budget of at least R$100–250/day
  lets the strategy learn (2–5 conversions/day).
- State budget per day and currency, and point out large values before creating.
