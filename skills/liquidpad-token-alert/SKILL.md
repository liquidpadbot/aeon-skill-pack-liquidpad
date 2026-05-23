---
name: LiquidPad Token Alert
description: Notify when a new token launches through LiquidPad and verify on-chain provenance.
var: ""
tags: [crypto, liquidpad, base]
---

> **${var}** — optional. Leave empty to alert on every new launch. Set to `kind:user` to filter (kinds: `user`, `platform`, `ecosystem`).

## What this does

LiquidPad ships dozens of tokens per week through its concept-curator flow (Telegram, CLI, MCP, X bot). Each launch:
- Deploys via `liquid-sdk` (open-source, MIT)
- Routes 80% creator / 15% $LPAD burn / 5% $LIQ buyback
- Gets ERC-8004 stamped (agent #50962)
- Becomes verifiable via the public `/api/verify/{address}` endpoint

This skill polls `/api/token-stats`, detects new addresses since the last run, calls `/api/verify` to confirm provenance, and sends a clean alert with the token page URL.

## Endpoints used

```
GET /api/token-stats        # all LiquidPad tokens with live USD market data
GET /api/verify/{address}   # confirms launch provenance, fee split, ERC-8004 id
```

Both CORS-enabled, no auth.

## Steps

1. Read `memory/logs/` (last 3 days) for the set of token addresses already alerted.
2. Fetch:
   ```bash
   curl -s "https://www.liquidpad.site/api/token-stats"
   ```
3. From `views.all[]`, walk newest-first by `deployedAt`. For each token:
   - Skip if `address` is in the already-alerted set.
   - If `${var}` is `kind:X`, skip tokens whose `kind` doesn't match.
4. For each new token:
   - Call `/api/verify/{address}` to fetch authoritative metadata:
     ```bash
     curl -s "https://www.liquidpad.site/api/verify/${address}"
     ```
   - Skip and log `_VERIFY_FAIL` if `verified !== true`.
5. Format alert (one entry per new token):
   ```
   🚀 new on LiquidPad

   NAME ($SYMBOL)
   {address}
   deployed: YYYY-MM-DD HH:MM UTC
   kind: user / platform

   market: price $X · MC $X · 24h vol $X
   page: liquidpad.site/t/{address}
   verify: liquidpad.site/api/verify/{address}
   ERC-8004: agent #50962

   fees: 80% deployer · 15% $LPAD burn · 5% $LIQ
   ```
6. Send batched notification via `./notify`.
7. Append all new addresses to `memory/logs/${today}.md` so subsequent runs don't re-alert.

## Sandbox note

Use **WebFetch** if outbound curl is blocked.

## Cost-saving knobs

- Default schedule is `*/30 * * * *` (every 30 min). For high-volume periods, the next-day digest (`liquidpad-stats-digest`) covers the same ground at lower frequency.
- Set `${var}` to `kind:user` to skip platform / ecosystem tokens.

## Errors

- `429` from `/api/token-stats`: backoff, retry next run.
- Empty `views.all[]`: log `LIQUIDPAD_TOKENS_EMPTY` (no recent launches).
- `/api/verify` returns `verified: false`: this token wasn't launched through LiquidPad — skip alert, log address for audit.

## Why this skill exists

If you're a builder on Liquid Protocol, you want to know fast which tokens launched via the verified LiquidPad path versus those someone deployed themselves and slapped the LiquidPad branding on. The verify endpoint is the anti-impersonation primitive.
