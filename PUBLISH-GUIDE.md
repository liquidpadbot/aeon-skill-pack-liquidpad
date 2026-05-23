# Publishing this skill pack

This file is for the maintainer (you). Not packaged into installs.

## 1. Create the GitHub repo

```bash
# from /root/liquidpad-web/aeon-skill-pack
git init
git add .
git commit -m "feat: initial skill pack — 4 skills (burn-monitor, stats-digest, token-alert, fee-tracker)"

# create the repo
gh repo create liquidpadbot/aeon-skill-pack-liquidpad --public --source=. --push --description "Aeon skill pack for LiquidPad — burn cycle alerts, token launch tracking, fee monitoring."

# (or use github.com UI to create the empty repo, then:)
git remote add origin https://github.com/liquidpadbot/aeon-skill-pack-liquidpad.git
git branch -M main
git push -u origin main
```

## 2. Verify install works (optional but recommended)

In a separate folder, fork Aeon and try installing:

```bash
git clone https://github.com/aaronjmars/aeon
cd aeon
./install-skill-pack liquidpadbot/aeon-skill-pack-liquidpad
```

The script should:
- Download the tarball
- Parse `skills-pack.json`
- Run security scanner against each `SKILL.md` (no HIGH findings expected — all skills only do `curl` + `./notify`)
- Copy into `skills/`
- Add disabled entries to `aeon.yml`

## 3. Open PR to Aeon README

Add a row to the `Community skill packs` table in `aaronjmars/aeon`:

```markdown
| aeon-skill-pack-liquidpad | 4 | Track $LPAD burn cycles, new LiquidPad token launches, protocol stats, and fee accruals on Base. |
```

PR template:
- Title: `docs: list aeon-skill-pack-liquidpad in community packs`
- Body: brief description, link to repo, mention pack passes manifest validation + security scan.

## 4. Tweet announcement

Tag @aeonframework + @aaronjmars. Keep it technical, no hype.

Example:
```
shipped: aeon-skill-pack-liquidpad

4 skills for @aeonframework users tracking LiquidPad on Base:

· burn-monitor — alert on every $LPAD buyback cycle
· token-alert — new launches, verified on-chain
· stats-digest — daily protocol overview
· fee-tracker — top-earning deployers

install: ./install-skill-pack liquidpadbot/aeon-skill-pack-liquidpad
```

Attach an OG image — can use the existing `https://www.liquidpad.site/api/og/burn` or `/api/og/manifesto`.

## 5. Maintenance

- Skills only call public endpoints (`/api/burn`, `/api/stats`, `/api/token-stats`, `/api/verify`) — no breaking changes expected from upstream Aeon.
- If Aeon's `skills-pack.json` schema evolves, bump `version` in `skills-pack.json` and update README compat note.
- Keep `default_enabled: false` for all skills — Aeon's spec strongly prefers explicit opt-in.
