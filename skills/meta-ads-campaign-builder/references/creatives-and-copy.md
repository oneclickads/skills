# Creatives and copy for Meta campaigns

## Creative sources

| Source                       | How                                                                                                                                          | Notes                                                                                                                                                   |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ad already in the account    | `getTopAdAccountCreatives`, `listCampaignAdsFromMeta`                                                                                        | Fastest for scaling. Keeps the ad's copy and URL tags.                                                                                                  |
| Image from a link            | public URL, one file per creative                                                                                                            | Optional centered crop to 9:16, 4:5 or 1:1 (images only).                                                                                               |
| Image in the media library   | `searchMediaLibraryImages` by file name                                                                                                      | No link needed.                                                                                                                                         |
| Video from a link            | `uploadVideoToMediaLibrary` (confirm first) → `getVideoUploadStatus` until ready                                                             | Meta processing takes minutes; create the ads only when it is ready.                                                                                    |
| Facebook Page post           | `findPagePostsFromMeta` by link, period, word or type                                                                                        | Runs the post as published, keeping likes and comments. Short share links do not identify the post: ask for the full link. Instagram posts are not supported. |
| Carousel                     | 2–10 cards, each with its own image or video, link, headline, description, optional button                                                   | Needs a Facebook Page on the ad; a card is image **or** video.                                                                                          |

### Ranking the best ads

Ask, or decide with the user:

- **Criterion**: return (ROAS/revenue), conversion volume, or cost per conversion.
- **Window**: last 7, 30 or 90 days (90 is the catalog default).
- **Consolidate by creative**: the same creative in several ad sets counts once.
- No revenue tracking → rank by conversions, then cost per conversion.
- An empty result after a slow Meta answer says nothing about the account: retry once,
  then shorten the range.

### Links

- The file must be shared as "anyone with the link". If it is restricted, the tool says so:
  ask the user to open sharing.
- A Google Drive folder is not a file. List it with a Drive tool if you have one;
  otherwise ask the user to connect Google Drive or send one link per file.
- A file on the user's computer does not work: they upload it somewhere and share it.

### Advantage+ creative enhancements

New image/video creatives go up with enhancements **off** unless the user asks. In
practice Meta only keeps "adapt to placement", on images. Generative features (text,
background, music, voice) are never turned on through the MCP.

### Placements

New creatives narrow each ad set to the placements that fit the asset's aspect ratio
(feed vs. vertical). Do not mix vertical and feed media in one ad set. Report the
placement result from `getWorkerStatus`; when it says it was not evaluated, say "not
checked", never "no incompatible placements".

## Copy fields

Primary text, headline, description, button (call to action), destination link, URL tags,
display link.

- **Inheritance**: each new ad set inherits copy from the reference ad of the ad set it
  copies, field by field. Changing only the headline keeps the rest.
- **URL tags**: not inherited on a new creative. Kept when reusing an existing ad. Ask
  "do you use tracking tags (UTMs)?" and apply them to every new ad. OneClick Tracking
  attributes sales by UTM, so a missing tag loses attribution.
- **Text options**: up to 5 primary texts, 5 headlines and 5 descriptions per new
  creative; Meta mixes them. Not for lead form ads.
- **Destination link**: required on every ad with media, except WhatsApp ads (button
  "Send WhatsApp message", number from the Page's connected WhatsApp). If the reference has
  no link, ask for it before creating.
- **Facebook Page**: the ads publish from the reference's Page; if it has none, ask which
  Page to use.
- **Lead forms**: the reference ad's form is reused unless the user picks another one from
  `listPageLeadGenFormsFromMeta`; a chosen form means `createCampaign`.

### Writing variations

When the user asks for variations, base them on what converts in the account (read the
best ads' copy with `getCreativeCopyFromMeta`), keep the offer and claims the user gave you,
and present them for approval before creating. Do not invent prices, discounts,
guarantees or results.
