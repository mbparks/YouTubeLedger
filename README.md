# View Ledger

A single-file, browser-based analytics tool for any public YouTube channel. Enter a free YouTube Data API key and a channel, and View Ledger pulls every public upload, charts viewership, reads the channel's momentum, breaks down content performance, and roughs out estimated revenue and profit. No build step, no server, no account.

Built as a Green Shoe Garage Data Co. instrument.

## What it does

View Ledger fetches a channel's entire public catalog through the YouTube Data API v3, then presents it across five tabs:

- **Trends.** Headline stats (including channel age, views per year, posting cadence, and time since last upload), a plain-language momentum verdict that accounts for dormancy, and four charts: per-video reach over time, reach by cohort, upload cadence, and engagement rate.
- **Catalog.** A views-per-video bar chart and a full sortable table of every video, with thumbnails, view counts, views-per-day, length, estimated revenue, likes, comments, and publish date. Badges mark Shorts, livestreams, made-for-kids, and AI/synthetic videos. Exports to CSV.
- **Content.** Tag-to-views analysis, performance by content type, category mix, caption and quality coverage, a channel-record reconciliation, and the channel's topics and keywords.
- **Revenue & profit.** An adjustable economics model with separate RPM settings for long-form and Shorts, a country-driven default RPM, and per-video cost inputs that net everything down to estimated profit.
- **Tracking.** Save a snapshot today, reload it on a later run, and the tool computes true growth in views and subscribers between the two dates.

There is also a one-click printable one-page summary.

## Getting started

### 1. Get a YouTube Data API key

The key is free and takes about two minutes.

1. Open the [Google Cloud Console](https://console.cloud.google.com/apis/library/youtube.googleapis.com).
2. Enable the "YouTube Data API v3" for a project.
3. Go to Credentials, create an API key, and copy it.

An unrestricted key works out of the box. If you restrict the key by HTTP referrer, allow the page where you open this tool.

### 2. Open the tool

Download `view-ledger.html` and open it in any modern browser. That is the whole installation. It runs entirely client-side, so you can open it straight from disk (a `file://` path) or host it as a static page anywhere (GitHub Pages, Netlify, an S3 bucket, and so on).

### 3. Run it

1. Paste your API key.
2. Enter a channel as a 24-character channel ID (`UC...`), an `@handle`, or a full channel URL.
3. Click "Pull & analyze."

The tool resolves the channel, walks its uploads playlist, and batch-fetches statistics. Larger catalogs take a few seconds and a handful of API calls.

## Using each tab

### Trends

The momentum verdict runs in two stages. First it reads activity: it measures days since the last upload and the median gap between uploads, then flags the channel as dormant when the silence runs past roughly 2.5 times its normal cadence (with a six-month floor), or slowing at about 1.5 times. A dormant channel reads "Dormant," never "Gaining momentum."

Second, for active channels, it gauges reach by comparing the newest third of the catalog against the oldest third by median lifetime views, restricted to videos at least 90 days old. It deliberately does not compare views-per-day across age cohorts, because views are front-loaded and then taper, so a newer video divided by fewer days looks inflated even when it performed no better than an older one. Comparing total lifetime views among matured videos sidesteps that bias.

The per-video reach chart still plots each video by publish date against its views-per-day on a logarithmic axis, with a dashed long-run trend line and a solid 3-month rolling average, because the bias is visible and harmless there. Videos under 30 days old appear as hollow dots and are excluded from the trend fit. The cohort table, cadence chart (with typical gap, last upload, and longest silence), and engagement chart round out the picture.

### Catalog

Toggle the bar chart between chronological order and most-viewed order. The table below is sortable on every column and shows a thumbnail per video. Badges flag Shorts (S), livestreams (LIVE), made-for-kids (KIDS), and AI or synthetic content (AI). The "Est. $" column reflects the RPM and Shorts settings from the Revenue & profit tab. Use "Export CSV" for the full dataset, including content type, category, caption status, definition, synthetic flag, tags, and per-video revenue, cost, and profit.

### Content

This tab is about what the channel makes and which choices correlate with reach. The tag chart ranks the uploader's tags by the median views of videos carrying them, limited to tags used on at least two videos. The content-type table separates Standard uploads, Shorts, and livestreams or premieres, since each format performs differently. The category mix maps YouTube's category IDs to names and shows share and median views per category. The captions and quality section reports the share captioned and in HD, then compares median views with and without captions. The channel-record reconciliation cross-checks the fetched catalog against YouTube's all-time view and video totals, so a gap from deleted or private videos is explained rather than mysterious. Topics and keywords show the channel's inferred topic categories and its own keyword tags.

A note on these signals: tag-to-views and caption-to-views are correlations, not proof of cause. A tag that rides popular videos may reflect the topic rather than the tag itself.

### Revenue & profit

Set five values:

- **Long-form RPM.** Dollars kept per 1,000 views on regular videos, after YouTube's cut.
- **Shorts RPM.** The same figure for Shorts, which typically monetize at a small fraction of long-form.
- **Shorts cutoff.** The duration in seconds at or below which a video is treated as a Short.
- **Cost per long-form video** and **cost per Short.** Your production cost per video, used to compute profit.

If the channel exposes a country, the long-form RPM is pre-set to a rough geography-based starting point (higher, mid, or lower RPM market). This is a coarse guess and is left alone once you type your own value. Livestreams are counted under the long-form RPM.

The tab shows gross revenue, estimated revenue per year (using channel age), total cost, net profit, average profit per video, and margin, a stacked revenue-by-year chart (long-form versus Shorts), and a per-video economics table ranked by profit. Costs default to zero, so profit equals revenue until you enter your own numbers.

### Tracking

A single pull is only a point-in-time reading of lifetime totals, so it cannot show real history on its own. To track genuine growth:

1. Pull a channel and click "Save snapshot" to download a JSON file.
2. Come back in a week or a month, pull the same channel again, and load the saved snapshot.

The tool reports views and subscribers gained, the daily rates, the marginal subscribers earned per new video published, and the biggest movers. Run it on a schedule and the snapshots become a real longitudinal record.

## How the estimates work

| Metric | Definition |
| --- | --- |
| Views-per-day | Lifetime views divided by days since publish |
| Activity status | Days since last upload versus the channel's median posting gap |
| Momentum (reach) | Median lifetime views of the newest third versus the oldest third, among videos at least 90 days old |
| Channel age | Time since the channel's creation date |
| Views per year | Total catalog views divided by channel age |
| Subs per video (cumulative) | Total subscribers divided by total uploads |
| Subs per new video (marginal) | Subscribers gained divided by videos published, between two snapshots |
| Tag performance | Median views of videos carrying each tag (minimum two videos) |
| Estimated revenue | Views multiplied by RPM, divided by 1,000, using a separate RPM for Shorts |
| Estimated profit | Estimated revenue minus your per-video cost |

## Limitations and honesty notes

These are structural to the public API, not bugs.

- **Public uploads only.** Private, unlisted, and deleted videos are not retrievable. This is why the Content tab reconciles the fetched catalog against YouTube's all-time totals, and the two will rarely match exactly.
- **Subscriber counts are rounded** by YouTube and may be hidden on some channels.
- **Revenue and profit are estimates, not earnings.** Real ad revenue is private to the channel owner (it lives behind the YouTube Analytics API and OAuth) and is never exposed publicly. The figures here exclude sponsorships, memberships, merchandise, and affiliate income, so treat them as back-of-envelope numbers and calibrate the RPM to the channel's niche.
- **The country-based RPM default is coarse.** It is a rough tier guess from the channel's country, nothing more.
- **Made-for-kids videos** are flagged but still estimated at full RPM, so revenue for a kids-heavy channel runs high, because those videos carry no personalized ads.
- **Shorts detection is a duration heuristic.** There is no official "is a Short" flag in the API. YouTube now allows Shorts up to roughly three minutes, so raise the cutoff (for example to 180 seconds) if a channel posts longer Shorts. Note also that YouTube changed Shorts view counting on March 31, 2025, so a Shorts view now counts each play or replay with no minimum watch time, which inflates Shorts views relative to long-form.
- **Livestream detection** keys off live timestamps and so also catches premieres. Concurrent live viewership is not retrievable after a broadcast ends.
- **Tag and caption correlations are not causal.** They describe what high-performing videos happen to share, not what made them perform.
- **Dislikes are not available.** YouTube made `dislikeCount` private in December 2021.

## Privacy

Your API key stays in the browser and is sent only to Google's API. Nothing is stored, logged, or transmitted anywhere else. There is no analytics, no tracking, and no backend. Snapshots are plain JSON files saved to your own machine, under your control.

## Tech stack

- A single self-contained HTML file (markup, styles, and vanilla JavaScript in one document).
- [Chart.js](https://www.chartjs.org/) loaded from a CDN for all charts.
- Fraunces and IBM Plex Mono web fonts from Google Fonts.

No frameworks, no package manager, and no build process. The only runtime dependencies are the CDN-hosted chart library and fonts, which require an internet connection when the page loads.

## API quota

The tool uses these endpoints: `channels.list` (once), `videoCategories.list` (once), `playlistItems.list` (paginated, 50 items per call), and `videos.list` (batched, 50 ids per call). A channel of about 150 videos costs on the order of 8 quota units. The default daily quota is 10,000 units, so even very large channels are inexpensive to analyze.

## Project structure

```
view-ledger.html    The entire application
README.md           This file
```

## Changelog

- **v8.** Added the Content tab: tag-to-views analysis, performance by content type, category mix, caption and quality coverage, channel-record reconciliation, and channel topics and keywords. Added a country-driven default RPM, livestream and synthetic-media detection, and richer CSV export.
- **v7.** Three no-extra-cost additions from data already fetched: channel age and views per year, video thumbnails and channel avatar, and a made-for-kids flag feeding the revenue estimate.
- **v6.** Reworked the momentum verdict to detect dormant and slowing channels from posting cadence, and rebased the reach comparison on median lifetime views to remove the views-per-day age bias that made dormant channels read as trending up.
- **v5.** Split the revenue model into separate RPMs for long-form and Shorts (duration-based detection), and added a cost side that nets revenue down to per-video profit.
- **v4.** Added the Revenue tab with an adjustable RPM, a revenue-by-year chart, and top earners. Added a 3-month rolling average to the engagement chart and a subscribers-per-video metric.
- **v3.** Rebranded to Green Shoe Garage Data Co. with a green palette, added subscriber tracking through snapshots, a 3-month rolling average on the velocity chart, and a printable one-page summary.
- **v2.** Added the Trends tab (momentum verdict, velocity chart, cohort comparison, upload cadence, engagement) and the Tracking tab for true longitudinal snapshots.
- **v1.** Initial release: pull every public video on a channel and chart views per video, with a sortable table and CSV export.

## License

GPL-3.0
