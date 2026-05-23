---
name: LiquidPad Fee Tracker
description: Track claimable trading fees on every LiquidPad-launched token. Surface top earners.
var: ""
tags: [crypto, liquidpad, base]
---

> $var — optional. Set to a 0x address to track only one wallet's launched tokens. Empty = top 10 across all LiquidPad tokens.

## What this does

Every token launched through LiquidPad routes 80% of trading fees to the deployer. Fees accrue inside Liquid Protocol's FeeLocker (permanent, locked LP) and need to be claimed to land in the deployer's wallet — they don't auto-stream.

This skill pulls the LiquidPad token roster (already enriched with live USD volume by /api/token-stats), and surfaces the 10 tokens with the most claimable fees right now. Useful if you're a deployer (wake-up call to claim) or an observer (sees who's actually earning).

## Endpoint used

```
GET https://www.liquidpad.site/api/token-stats
```

Returns views.all containing address, name, symbol, volume24hUsd, deployedAt, and other fields.

For exact claimable amounts, deployers can use liquid-sdk directly via getFees, but this skill estimates from 24h volume rather than calling the SDK on every token — it's a daily digest, not a precise accountant.

## Steps

1. Fetch the full token list:
```
curl -s https://www.liquidpad.site/api/token-stats
```

2. From views.all compute estimated fees-per-token:
   - 1% trading fee × 80% creator share = 0.8% of 24h volume goes to deployer
   - For each token: estDeployerEarn24h = volume24hUsd * 0.008

3. Sort by estDeployerEarn24h desc.

4. If $var is a 0x address, filter to tokens deployed by that wallet (requires /api/token-stats to expose a deployer field — fall back to showing all if missing).

5. Take top 10 (or all if fewer than 10 have non-zero volume).

6. Format:
   ```
   LiquidPad fee tracker — top earners (last 24h, est)

   1. NAME ($SYMBOL) · ~$X.X earned · $X.XK vol
      liquidpad.site/t/0x...
   2. ...

   total est across top 10: ~$X.X
   note: estimate — actual claimable may differ. Use liquid-sdk getFees for exact amounts.
   ```

7. Send via ./notify.

8. Log to memory/logs/ for the current date.

## Why estimate, not query SDK

Querying getFees per-token costs 1 RPC call each. For 100+ tokens, that's 100+ RPC calls every run. Volume-derived estimate is good enough for "where's the action" reporting.

If you want exact numbers for one specific token, run a one-off liquid-sdk call against your RPC.

## Sandbox note

Use WebFetch if outbound curl is blocked.

## Pairing

- liquidpad-token-alert — get notified on new launches (then this skill tracks their earning curve)
- liquidpad-burn-monitor — see where the 15% goes
- liquidpad-stats-digest — daily protocol-wide overview
