---
name: LiquidPad Stats Digest
description: Daily LiquidPad protocol digest — tokens deployed, momentum, top by 24h volume.
var: ""
tags: [crypto, liquidpad, digest]
---

> $var — optional. Leave empty for full daily digest. Set to `top:5` to limit top-volume list to N tokens.

## What this does

Once a day, pull a snapshot of LiquidPad's protocol-wide metrics and post a clean digest. Useful if you're tracking the launchpad's growth, comparing weekly momentum, or staying aware of which user-launched tokens are moving real volume.

Same data the liquidpad.site/stats page renders, but delivered to Telegram / Discord / Slack on a schedule, no browser tab needed.

## Endpoint used

```
GET https://www.liquidpad.site/api/stats
```

Returns totals (tokens, liveTokens, volume24hUsd, marketCapUsd), momentum (last7d, prior7d, change7dPct), timeline (firstDeployAt, lastDeployAt), daily7 array, topByVolume array, asOf timestamp.

CORS-enabled, no auth required.

## Steps

1. Fetch:
```
curl -s https://www.liquidpad.site/api/stats
```

2. Parse top-level fields. If $var matches `top:N`, slice topByVolume to N entries; otherwise default to top 5.

3. Format the digest:
   ```
   LiquidPad daily digest — YYYY-MM-DD

   tokens deployed: 189 total (139 this week, vs 50 prior — +178%)
   24h aggregate volume: $6.7K across 12 live tokens

   top by 24h vol:
     1. NAME ($SYMBOL) · $X.XK vol · liquidpad.site/t/0x...
     2. ...

   first deploy: 2026-04-15
   latest: 2026-05-23

   protocol page: liquidpad.site/stats
   ```

   (Numbers above are illustrative — your skill output will reflect the live response from /api/stats.)

4. Send via ./notify.

5. Log raw payload to memory/logs/ for the current date for trend analysis by other skills (e.g. weekly review).

## Sandbox note

Use WebFetch if outbound curl is blocked.

## Errors

- /api/stats returns an error field: skip digest, log error, do NOT notify.
- Empty topByVolume: still send digest with totals + momentum, omit top section.

## Pairing

This skill pairs naturally with liquidpad-burn-monitor (alert-on-event) and liquidpad-fee-tracker (claim reminders) — together they give you a complete view of LiquidPad activity without ever opening the app.
