# Google Ads campaign types — structure and practice

## Search

**What to set**: ad groups, each with keywords (exact, phrase, broad) and responsive search
ads (up to 15 headlines of 30 characters, up to 4 descriptions of 90); headlines and
descriptions can be pinned; negative keywords; final URL. Optional: Search Partners,
Display Expansion, sitelinks, callouts, structured snippets, call, images, prices,
promotions, lead form. Bidding: maximize clicks, maximize conversions (optional target
CPA), maximize conversion value (optional target ROAS), manual CPC, target impression
share.

**Practice**:

- Separate **brand** and **non-brand** into different campaigns: brand is cheap and
  protects the name; non-brand needs its own budget and targets.
- One theme per ad group (one intent, closely related keywords) so ad and landing page
  match the search.
- Start with exact and phrase match; add broad match once conversion bidding has data.
- Negatives from day one: "grátis", "gratuito", "download", "pdf", "vagas", "emprego",
  "curso" (unless they sell courses), "reclame aqui", unwanted competitor terms. Review the
  search terms report weekly and add more.
- Fill the RSA: 10–15 headlines and 4 descriptions, keyword in several headlines; pin the
  brand to position 1 only when needed (pinning cuts combinations).
- At least 4 sitelinks and 4 callouts.
- No conversion history: maximize clicks (optionally capped) first, then maximize
  conversions once conversions come in steadily (about 15–30 a month).
- Search Partners and Display Expansion off unless the user wants more reach.

## Performance Max

**What to set**: business name and logos; asset groups (headlines, long headlines,
descriptions, landscape, square and portrait images, YouTube videos, final URL, call to
action, search themes, audience signal); optional sitelinks and other assets, brand
exclusions, new-customer goal, Merchant Center feed (retail PMax). Bidding: maximize
conversions (optional target CPA) or maximize conversion value (optional target ROAS).
Needs conversion tracking.

**Practice**:

- One asset group per product line or audience theme, each with its own landing page.
- Fill every asset slot; include at least one real video, or Google auto-generates a weak
  one.
- Search themes and first-party audiences (customer lists, site visitors) as signals.
- When a brand Search campaign exists, exclude the brand with a brand list so PMax does
  not take credit for brand searches.
- Give it 4–6 weeks before judging; avoid frequent edits that reset learning.

## Demand Gen

**What to set**: ad groups with location, language, optional audience and channel
selection (YouTube incl. Shorts, Discover, Gmail, Display); image, video or carousel ads;
optional Merchant Center products. Bidding: maximize conversions, maximize conversion value,
maximize clicks. Google enforces a minimum daily budget.

**Practice**: test an ad group with an audience against one optimized by Google; supply
vertical video for Shorts and landscape plus square images; judge it as prospecting, on
cost per conversion over weeks.

## Display

**What to set**: ad groups with targeting (audiences, interests, topics, placements,
YouTube channels, contextual keywords) and responsive display ads (headlines, long
headline, descriptions, business name, images, logo). Bidding: manual CPC, viewable CPM,
maximize clicks, conversions or conversion value.

**Practice**: best for remarketing site visitors and cheap reach; exclude mobile app
placements if traffic is junk; cap frequency for remarketing.

## Shopping

**What to set**: a Merchant Center account linked to Google Ads, campaign priority, product
groups (all products, or split by brand, category, product type or item id with an
"everything else" group). Bidding: manual CPC per product group, maximize clicks, target
ROAS, maximize conversion value. Language comes from the feed.

**Practice**: start with all products, then split best sellers or high-margin groups to
bid them separately; a clean feed (titles, images, prices, GTINs) matters more than bids.

## App

**What to set**: subtype (installs, engagement, or pre-registration on Google Play; it
cannot change later), the app, app conversion actions, texts, images and videos that
Google combines.

**Practice**: installs need an in-app event to optimize toward quality users once
volume allows (target CPA on that event).

## Not creatable through the API

Smart campaigns (Google no longer lets integrations create them) and Video campaigns
(created only in Google Ads; use Demand Gen or PMax for YouTube video through an API).

## Budget sanity

- The daily budget is an average; Google can spend up to 2× on a given day while keeping
  the monthly average (about 30.4 × daily).
- Tie budget to targets: with a target CPA of R$50, at least R$100–250/day lets the
  strategy learn (2–5 conversions/day).
- State budget per day and currency; point out large values.
