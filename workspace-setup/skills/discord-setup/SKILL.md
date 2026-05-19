---
name: discord-setup
description: Step-by-step procedure for setting up Discord on an OpenClaw instance. Trigger on any request to connect, set up, or configure Discord.
---

# Discord Setup

Read the full runbook before starting:
```
https://raw.githubusercontent.com/almnjoy/madeformeai-private-runbooks/main/discordsetup.md
```

## Steps (summary — runbook is authoritative)

**Step 1 — Install plugin (bundled, no npm needed)**
```bash
openclaw plugins install discord
```
Discord is a bundled plugin. Do NOT install `@openclaw/discord` via npm — that is a different package that conflicts with the bundled version.

**Step 2 — Ask user for Discord bot token**
Tell them: Discord Developer Portal → Applications → New Application → Bot → Reset Token → copy it.
Also need: Bot → enable Message Content Intent, Server Members Intent, Presence Intent.

**Step 3 — Write token to config using Python3 ONLY**
```bash
python3 - <<'PY'
import json
p = '/home/node/.openclaw/openclaw.json'
with open(p) as f: cfg = json.load(f)
cfg.setdefault('channels', {}).setdefault('discord', {})
cfg['channels']['discord']['token'] = 'TOKEN_HERE'
cfg.setdefault('plugins', {}).setdefault('entries', {}).setdefault('discord', {})
cfg['plugins']['entries']['discord']['enabled'] = True
with open(p, 'w') as f: json.dump(cfg, f, indent=2)
print('done')
PY
```

**Step 4 — Restart gateway**
```bash
kill $(pgrep -f 'node.*dist/index.js') 2>/dev/null; true
```
Wait 20–30 seconds.

**Step 5 — Verify**
```bash
openclaw status --deep
```
Success: `Discord │ ON │ OK │ configured`

**Step 6 — Invite bot to server**
Tell user: use the OAuth2 URL from Discord Developer Portal to invite the bot to their server.

## Critical: never install @openclaw/discord via npm

The bundled `discord` plugin is the correct one. Installing the npm package on top breaks both.
If `@openclaw/discord` was accidentally installed:
```bash
rm -rf /home/node/.openclaw/npm/node_modules/@openclaw/discord
```
Then reinstall bundled and restart.
