# Change rules for existing Meta structures

Money values are in the **ad account currency, not cents** (500 = R$500). Every write
tool below is live in Meta; bundle the changes to one object into one call and leave 30 s
between writes to the same campaign, ad set or ad (every `update*` tool counts).

## What each change costs

| Change                                              | Tool                               | Cost to tell the user before confirming                                                                       |
| --------------------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Audience (location, age, gender, interests, custom audiences, Advantage+) | `updateAdSetAudience` | Can restart Meta's learning phase: unstable for days, may get worse. Suggest a new ad set instead. No undo in Meta. |
| Placements                                          | `updateAdSet` `placements`         | Can restart learning. Suggest a new ad set instead. Moving off automatic placements is reported.              |
| Text, headline, description, link, display link, button | `updateAdCopy`                 | Meta reviews the ad again; the ad set may re-enter learning; a new post is published, likes and comments stay on the old one. |
| URL tags only                                       | `updateAdCopy` `urlTags` alone     | Meta reviews the ad again; the post, likes and comments are kept.                                             |
| Creative swap                                       | `updateAd` `creativeId`            | Meta reviews the ad again; likes and comments stay on the old post. Ad keeps its id and history.              |
| Bid or cost cap amount                              | `updateAdSet` `bidAmount`          | A cap too low practically stops delivery.                                                                     |
| Turn on, raise budget, raise a cap or limit         | `update*`, `setMetaAdAccountMonthlyBudget` | Spends more money. Point out large jumps.                                                             |
| Status off, rename                                  | `updateCampaign` / `updateAdSet` / `updateAd` | A name is replaced whole, not search-and-replace.                                                  |

The tools state no learning-phase effect for budget, status, name, schedule or caps: do not
claim one either way.

## Campaign — `updateCampaign`

| Field            | Rule                                                                                                                  |
| ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| `status`         | ACTIVE / PAUSED. Does not change ad sets or ads paused on their own.                                                  |
| `name`           | Replaces the whole name. Keep OneClick's suffixes and the 🤖 [OneClick ON] marker unless the user asks otherwise.      |
| `dailyBudget`    | CBO only (read `budgetOptimization` with `getCampaignFromMeta`). At least Meta's per-currency minimum. Refused on a campaign with a lifetime budget. |
| `lifetimeBudget` | CBO on a lifetime budget (`lifetimeBudget` above 0). Spent until its ad sets end: move the end with each ad set's `endTime`. |
| `spendCap`       | Total the campaign may spend; `0` removes it.                                                                         |

Meta refuses switching a campaign between daily and lifetime budget.

## Ad set — `updateAdSet`

| Field                 | Rule                                                                                                                       |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `dailyBudget`         | ABO only. On an ABO ad set running on a lifetime budget it switches it back to daily.                                     |
| `lifetimeBudget`      | ABO ad set; needs an end time; replaces a daily budget.                                                                    |
| `dailyMinSpendTarget` | CBO with a daily budget; `0` removes it. Meta refuses any write, activation included, that takes the ad sets' minimums above the campaign budget: lower them first. |
| `dailySpendCap`       | CBO; the most the campaign spends on this ad set per day; `0` removes it.                                                  |
| `bidAmount`           | Only on COST_CAP or LOWEST_COST_WITH_BID_CAP (read with `getAdSetFromMeta`; in CBO the strategy is the campaign's, the amount is still set here). Changes the amount, never the strategy. |
| `startTime`/`endTime` | `YYYY-MM-DDTHH:mm`, no offset: read in the ad account's time zone, which the result names. Tell the user when it may not be theirs. End after start. |
| `placements`          | Replaces the whole list: read the current one, send every token that stays. `platform:*` keeps a whole platform; `['*']` alone returns to automatic (Advantage+). A retired placement is kept if already running, refused otherwise. An ad whose format does not fit a kept placement does not run there. |

## Audience — `updateAdSetAudience`

Read with `getAdSetFromMeta` first. Send only what the user asked to change: an omitted
dimension is kept, and clearing is always explicit.

| Dimension                      | Rule                                                                                                                        |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `geoLocations`                 | Replaces the whole geography; cannot be cleared. Region/city/zip keys from `searchAdGeoLocationsFromMeta` (show matches by name, ask when several fit). A place inside a country already targeted is refused: send the place instead of the country. Pins (`customLocations`): never guess coordinates; current pins sit under `geoLocationsUnmapped.custom_locations` and must be resent to keep them. |
| `excludedGeoLocations`         | Replaces the whole exclusion; `{}` removes every exclusion.                                                                 |
| `ageMin` / `ageMax`            | 13–65 (65 = 65+); one end alone keeps the other. With Advantage+ on, Meta refuses `ageMin` above 25 and requires `ageMax` 65. |
| `genders`                      | `[]` reaches all genders.                                                                                                   |
| Custom audiences               | CUSTOM rows of `listAdAccountAudiencesFromMeta` with `operationStatus` 200, offered by name with size and subtype. Use `addCustomAudienceIds` / `removeCustomAudienceIds` (and the excluded pair) for one audience. When the ad set already has some and the user picks one, ask: join or replace all? A SAVED audience is refused. |
| Interests                      | `addInterestIds` (to the first detailed-targeting group, widens reach) / `removeInterestIds` (from every group; the last one removed leaves the ad set broad). Ids from `searchAdInterestsFromMeta`, shown by name with size and path. |
| `advantageAudience`            | 1 lets Meta deliver beyond the audience; 0 keeps it inside. Set only on the user's request, never to make another change go through. 1 needs an open age range: tell the user, let them decide, never widen it yourself. Some objectives require it on. |

Not editable here: placements (use `updateAdSet`), behaviors, demographics, unmapped
geography.

## Ad — `updateAd` and `updateAdCopy`

- `updateAd`: `status`, `name`, `creativeId` (another creative of the same account, from
  `listCampaignAdsFromMeta`). Delivers only when ad set and campaign are ACTIVE.
- `updateAdCopy`: show the current copy with `getCreativeCopyFromMeta` first.
  - `link` goes without tracking parameters; those go in `urlTags`.
  - `urlTags` replaces the whole string: read it and keep `{{ad.id}}`.
  - On a video ad with no button, send `callToActionType` with `link` (and with
    `displayLink`).
  - A field with several text options is refused listing them; show them and ask. Send
    `replaceTextOptions` only after the user chose to collapse them into one.
  - Refused with nothing changed: dynamic creative, catalog ad, carousel; text, link or
    button on an ad built from an existing post; the link of an ad whose button opens an
    Instant Form.
  - Report `creativeFeaturesChanged` and `replacedOptions`. `previousCreativeId` undoes the
    edit through `updateAd`.

## Account monthly limit — `setMetaAdAccountMonthlyBudget`

- OneClick's ceiling for the account's month, not a Meta budget and not per day: OneClick
  derives a daily reference (about the monthly value ÷ days in the month) and stops raising
  budgets or duplicating on optimized campaigns as spend approaches it. It never forces
  spend and never touches campaigns it does not optimize.
- Takes OneClick's internal numeric account id, not the `act_` id (the tool's description
  names where to read it).
- `monthlySpendMax: null` removes it. Removing is rejected while an active optimization
  has it as its only ceiling; the error names the campaigns, which first need a daily
  maximum or unlimited scale (skill `optimization-setup`).
- Confirm the exact value and currency before writing.
