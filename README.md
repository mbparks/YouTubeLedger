# View Ledger

A single-file, browser-based analytics tool for any public YouTube channel. Enter a free YouTube Data API key and a channel, and View Ledger pulls every public upload, charts viewership, reads the channel's momentum, and roughs out estimated revenue and profit. No build step, no server, no account.

Built as a Green Shoe Garage Data Co. instrument.

## What it does

View Ledger fetches a channel's entire public catalog through the YouTube Data API v3, then presents it across four tabs:

- **Trends.** Headline stats, a plain-language momentum verdict (gaining, holding, or cooling), and four charts: per-video reach over time, reach by cohort, upload cadence, and engagement rate.
- **Catalog.** A views-per-video bar chart and a full sortable table of every video, with view counts, views-per-day, length, estimated revenue, likes, comments, and publish date. Exports to CSV.
- **Revenue & profit.** An adjustable economics model with separate RPM settings for long-form and Shorts, plus per-video cost inputs that net everything down to estimated profit.
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

The momentum verdict splits the catalog into three equal date ranges and compares the most recent third against the oldest by median views-per-day. Views-per-day (views divided by days online) is used instead of raw views because raw totals always flatter older videos that have simply been up longer.

The per-video reach chart plots each video by publish date against its views-per-day on a logarithmic axis. A dashed line shows the long-run fitted trend, and a solid line shows a 3-month rolling average. Videos under 30 days old appear as hollow dots and are excluded from the trend fit because they are still accumulating views.

The cohort table, cadence chart, and engagement chart round out the picture. Engagement is likes as a percentage of views, also drawn with a 3-month rolling average.

### Catalog

Toggle the bar chart between chronological order and most-viewed order. The table below is sortable on every column. The "Est. $" column reflects the RPM and Shorts settings from the Revenue & profit tab, and a small "S" badge marks any video detected as a Short. Use "Export CSV" for the full dataset, including per-video revenue, cost, and profit.

### Revenue & profit

Set five values:

- **Long-form RPM.** Dollars kept per 1,000 views on regular videos, after YouTube's cut.
- **Shorts RPM.** The same figure for Shorts, which typically monetize at a small fraction of long-form.
- **Shorts cutoff.** The duration in seconds at or below which a video is treated as a Short.
- **Cost per long-form video** and **cost per Short.** Your production cost per video, used to compute profit.

The tab then shows gross revenue, total cost, net profit, average profit per video, and margin, a stacked revenue-by-year chart (long-form versus Shorts), and a per-video economics table ranked by profit. Costs default to zero, so profit equals revenue until you enter your own numbers.

### Tracking

A single pull is only a point-in-time reading of lifetime totals, so it cannot show real history on its own. To track genuine growth:

1. Pull a channel and click "Save snapshot" to download a JSON file.
2. Come back in a week or a month, pull the same channel again, and load the saved snapshot.

The tool reports views and subscribers gained, the daily rates, the marginal subscribers earned per new video published, and the biggest movers. Run it on a schedule and the snapshots become a real longitudinal record.

## How the estimates work

| Metric | Definition |
| --- | --- |
| Views-per-day | Lifetime views divided by days since publish |
| Momentum | Median views-per-day of the newest third versus the oldest third |
| Subs per video (cumulative) | Total subscribers divided by total uploads |
| Subs per new video (marginal) | Subscribers gained divided by videos published, between two snapshots |
| Estimated revenue | Views multiplied by RPM, divided by 1,000, using a separate RPM for Shorts |
| Estimated profit | Estimated revenue minus your per-video cost |

## Limitations and honesty notes

These are structural to the public API, not bugs.

- **Public uploads only.** Private, unlisted, and deleted videos are not retrievable. Shorts are included.
- **Subscriber counts are rounded** by YouTube and may be hidden on some channels.
- **Revenue and profit are estimates, not earnings.** Real ad revenue is private to the channel owner (it lives behind the YouTube Analytics API and OAuth) and is never exposed publicly. The figures here exclude sponsorships, memberships, merchandise, and affiliate income, so treat them as back-of-envelope numbers and calibrate the RPM to the channel's niche.
- **Shorts detection is a duration heuristic.** There is no official "is a Short" flag in the API. YouTube now allows Shorts up to roughly three minutes, so raise the cutoff (for example to 180 seconds) if a channel posts longer Shorts.
- **Velocity favors evergreen older content,** which can keep accumulating views for years. Read the momentum verdict as a quick take, and lean on the cohort table and snapshot comparison for the cleaner signal.

## Privacy

Your API key stays in the browser and is sent only to Google's API. Nothing is stored, logged, or transmitted anywhere else. There is no analytics, no tracking, and no backend. Snapshots are plain JSON files saved to your own machine, under your control.

## Tech stack

- A single self-contained HTML file (markup, styles, and vanilla JavaScript in one document).
- [Chart.js](https://www.chartjs.org/) loaded from a CDN for all charts.
- Fraunces and IBM Plex Mono web fonts from Google Fonts.

No frameworks, no package manager, and no build process. The only runtime dependencies are the CDN-hosted chart library and fonts, which require an internet connection when the page loads.

## API quota

The tool uses three endpoints: `channels.list` (once), `playlistItems.list` (paginated, 50 items per call), and `videos.list` (batched, 50 ids per call). A channel of about 150 videos costs on the order of 7 quota units. The default daily quota is 10,000 units, so even very large channels are inexpensive to analyze.

## Project structure

```
view-ledger.html    The entire application
README.md           This file
```

## License

GPL-3.0
