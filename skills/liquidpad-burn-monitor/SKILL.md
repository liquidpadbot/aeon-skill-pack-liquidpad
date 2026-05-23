---
name: LiquidPad Burn Monitor
description: Track $LPAD buyback & burn cycles. Alert when a new cycle ships.
var: ""
tags: [crypto, liquidpad, base]
---

> $var — optional. Leave empty to track all cycles. Set to `since:2026-05-01` to only report cycles after a date.

## What this does

LiquidPad runs a buy-and-burn loop on Base mainnet:
- 15% of every trading fee on every LiquidPad-launched token buys $LPAD on market
- $LPAD goes to 0x000000000000000000000000000000000000dEaD permanently
- 5% buys $LIQ (Liquid Protocol's token) as ecosystem support

This skill polls the public /api/burn endpoint, compares cumulative burn / cycle count to the last run, and alerts you on any new cycle so you don't have to babysit the Telegram channel.

LiquidPad is a third-party builder on Liquid Protocol's open-source primitives. Not affiliated with Liquid Protocol. Full attribution at liquidpad.site/built-on-liquid.

## Endpoint used

```
GET https://www.liquidpad.site/api/burn
```

Returns a JSON object with cycles array and totals object containing burned, liqBought, burnTxCount, liqBuyTxCount.

CORS-enabled, no auth, public endpoint. Cached 30s upstream.

## Steps

1. Read memory/MEMORY.md and memory/logs/ from the last 7 days for the previous burn snapshot.

2. Fetch the current ledger:
```
curl -s https://www.liquidpad.site/api/burn
```

3. Parse totals.burnTxCount and totals.burned. Compare to the last logged values.

4. If burnTxCount increased OR burned increased by at least 1%:
   - Identify the new cycles by walking cycles[] newest-first until you hit a txHash that's already in the log.
   - For each new cycle, format:
     ```
     LiquidPad burn cycle #N

     burned: X.XXM $LPAD to 0x...dEaD
     tx: 0xabc1234567... (basescan.org/tx/0xabc...)
     timestamp: YYYY-MM-DD HH:MM UTC

     cumulative: [running_total] $LPAD destroyed across [total_cycles] cycles
     [running_liq_total] $LIQ bought back as ecosystem support

     proof: liquidpad.site
     ```

5. Send a single notification via ./notify aggregating all new cycles found in this run.

6. Log current totals to memory/logs/ for the current date for next comparison.

7. If no new cycles, log LIQUIDPAD_BURN_OK and exit silent.

## Output

Markdown, one section per new cycle. Optimized for Telegram / Discord embed.

## Sandbox note

The sandbox may block outbound curl. Use WebFetch as a fallback for the /api/burn URL.

## Errors

- 502 / network from /api/burn: log LIQUIDPAD_BURN_UPSTREAM_FAIL, retry next run, do NOT alert.
- Empty cycles array: harmless. Protocol just rolled out, log LIQUIDPAD_BURN_EMPTY.

## Why this skill exists

LiquidPad is a self-sustaining agent on Base. Every fee fuels the next burn. Without this skill you would manually refresh liquidpad.site to know when a cycle shipped. With it your Aeon instance gives you a Telegram ping the second a tx lands. Same shape as Aeon's core token-alert skill, but scoped to a specific deflationary loop you actually care about.
