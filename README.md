# aeon-skill-pack-liquidpad

Aeon skill pack for monitoring [LiquidPad](https://www.liquidpad.site) — an independent, MIT-licensed launchpad on Base built on [Liquid Protocol](https://github.com/Liquid-Protocol-Ops)'s open-source primitives.

## Quick view

| You'll get | When | From |
|---|---|---|
| 🔥 Burn alert | new `$LPAD` buyback & burn cycle | `/api/burn` (every 6h) |
| 🚀 New token alert | every LiquidPad-deployed token, with on-chain provenance verified | `/api/token-stats` + `/api/verify` (every 30m) |
| 📊 Daily digest | tokens deployed, weekly momentum, top by 24h volume | `/api/stats` (daily 14:00 UTC) |
| 💰 Fee tracker | estimated claimable trading fees on every launched token | `/api/token-stats` (daily 08:00 UTC) |

All four skills run on **public, CORS-enabled, no-auth** endpoints — no API keys to provision, no rate-limit tokens to rotate.

> Independent third-party project. **Not affiliated with Liquid Protocol.** Full attribution at [liquidpad.site/built-on-liquid](https://www.liquidpad.site/built-on-liquid).
>
> Listed in aeon framework's [community-packs](https://github.com/aaronjmars/aeon/blob/main/community-packs.md) and [ECOSYSTEM.md](https://github.com/aaronjmars/aeon/blob/main/ECOSYSTEM.md).

## Install

In your Aeon repo:

```bash
./install-skill-pack liquidpadbot/aeon-skill-pack-liquidpad
```

This downloads the pack, runs Aeon's security scanner against each `SKILL.md`, copies skills into your `skills/` folder, and adds disabled entries to `aeon.yml`. Toggle them on from the Aeon dashboard.

### Verify install

Right after install, confirm the pack registered correctly:

```bash
grep liquidpad aeon.yml      # 4 entries, all enabled: false
ls skills/liquidpad-*         # 4 directories
cat skills.lock               # pack: LiquidPad provenance row
```

Run any single skill manually with `./aeon` once before scheduling — easiest sanity check that `/api/burn` and friends are reachable from your Actions sandbox.

## Skills

| Slug | Schedule | Category | What it does |
|------|----------|----------|--------------|
| `liquidpad-burn-monitor` | every 6h | crypto | Alert on new $LPAD buyback & burn cycles. |
| `liquidpad-token-alert` | every 30m | crypto | Notify on every new token launched through LiquidPad, with on-chain provenance verified. |
| `liquidpad-stats-digest` | daily 14:00 UTC | crypto | Daily protocol digest — tokens deployed, weekly momentum, top by volume. |
| `liquidpad-fee-tracker` | daily 08:00 UTC | crypto | Estimated claimable fees on every LiquidPad-launched token. Surface top earners. |

All schedules are defaults — configurable via `aeon.yml` after install.

## Endpoints used

Every skill in this pack consumes LiquidPad's **public, CORS-enabled, no-auth** endpoints:

| Endpoint | Used by |
|----------|---------|
| `GET /api/burn` | burn-monitor |
| `GET /api/stats` | stats-digest |
| `GET /api/token-stats` | token-alert · fee-tracker |
| `GET /api/verify/{address}` | token-alert (provenance check) |

No API key. No rate-limit token. No upstream dependency on LiquidPad's infrastructure beyond the cached read endpoints. Same data the website renders, delivered to your notification channel on a schedule.

## Why these skills

LiquidPad self-deploys to several surfaces (Telegram bot, CLI, MCP server, X bot). Aeon users — many of whom run their own self-funding agents — benefit from monitoring LiquidPad without keeping a browser tab open. Specifically:

- **Burn monitor** turns on-chain proof events into push notifications.
- **Token alert** combined with `/api/verify` is the anti-impersonation primitive — confirm a token actually launched through LiquidPad before signal-boosting it.
- **Stats digest** replaces the daily "wen LiquidPad update" check.
- **Fee tracker** answers "is anyone actually earning?" — useful sanity check for a deflationary launchpad.

## Compatible with

- Aeon (any version with `install-skill-pack` + `skills-pack.json` manifest support)
- Aeon's notification backends: Telegram, Discord, Slack, Email
- Aeon's MCP server / A2A gateway — these skills work the same when called from Claude Desktop, Claude Code, LangChain, AutoGen, CrewAI, OpenAI Agents SDK

## Related

- [LiquidPad MCP server](https://www.npmjs.com/package/liquidpad-mcp) — `npx -y liquidpad-mcp` for direct tool access from Claude / Cursor / Kiro
- [LiquidPad CLI](https://www.npmjs.com/package/liquidpad-cli) — `npm i -g liquidpad-cli` for command-line deploys
- [LiquidPad on Telegram](https://t.me/liquidpadbot) — concept curator + API key issuance
- [LiquidPad on X](https://x.com/LiquidPadBot) — pitch a vibe, ship a token
- [Liquid Protocol SDK](https://github.com/Liquid-Protocol-Ops/SDK) — the underlying primitive
- [Aeon framework](https://github.com/aaronjmars/aeon) — the autonomous agent framework this pack runs on

## License

MIT — see [LICENSE](./LICENSE).

Trademarks for "Aeon", "Liquid Protocol", and "$LIQ" belong to their respective owners. This pack references those names solely for accurate attribution and ecosystem interoperability.
