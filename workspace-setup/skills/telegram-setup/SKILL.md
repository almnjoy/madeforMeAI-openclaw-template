---
name: telegram-setup
description: Step-by-step procedure for setting up Telegram on an OpenClaw instance. Trigger on any request to connect, set up, or configure Telegram.
---

# Telegram Setup

Read the full runbook before starting:
```
https://raw.githubusercontent.com/almnjoy/madeformeai-private-runbooks/main/telegram-setup.md
```

## Steps (summary — runbook is authoritative)

**Step 1 — Install plugin (bundled, no npm needed)**
```bash
openclaw plugins install telegram
```

**Step 2 — Ask user for their Telegram bot token**
Tell them: open Telegram → search @BotFather → `/newbot` → copy the token.
Do not proceed until you have the token.

**Step 3 — Write token to config using Python3 ONLY**
```bash
python3 - <<'PY'
import json
p = '/home/node/.openclaw/openclaw.json'
with open(p) as f: cfg = json.load(f)
cfg.setdefault('channels', {}).setdefault('telegram', {})
cfg['channels']['telegram']['token'] = 'TOKEN_HERE'
cfg.setdefault('plugins', {}).setdefault('entries', {}).setdefault('telegram', {})
cfg['plugins']['entries']['telegram']['enabled'] = True
with open(p, 'w') as f: json.dump(cfg, f, indent=2)
print('done')
PY
```

Replace `TOKEN_HERE` with the actual token. Never use any other config write method.

**Step 4 — Restart gateway**
```bash
kill $(pgrep -f 'node.*dist/index.js') 2>/dev/null; true
```
Wait 20–30 seconds.

**Step 5 — Verify**
```bash
openclaw status --deep
```
Success: `Telegram │ ON │ OK │ configured`

**Step 6 — Pairing**
New Telegram users will need pairing approval. Tell the user to message the bot, then approve:
```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

## If token write fails

Stop. Do not retry with a different method. Report the error.

## Success state
```
Telegram │ ON │ OK │ configured · accounts 1/1
```
