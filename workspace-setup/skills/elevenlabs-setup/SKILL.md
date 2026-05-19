---
name: elevenlabs-setup
description: Step-by-step procedure for setting up ElevenLabs voice on an OpenClaw instance. Trigger on any request to add voice, text-to-speech, or ElevenLabs.
---

# ElevenLabs Setup

Read the full runbook before starting:
```
https://raw.githubusercontent.com/almnjoy/madeformeai-private-runbooks/main/elevenlabs-voice.md
```

## Steps (summary — runbook is authoritative)

**Step 1 — Install plugin (bundled)**
```bash
openclaw plugins install elevenlabs
```

**Step 2 — Ask user for ElevenLabs API key**
Tell them: elevenlabs.io → Profile → API Keys → copy key.

**Step 3 — Write config using Python3 ONLY**
```bash
python3 - <<'PY'
import json
p = '/home/node/.openclaw/openclaw.json'
with open(p) as f: cfg = json.load(f)
cfg.setdefault('plugins', {}).setdefault('entries', {}).setdefault('elevenlabs', {})
cfg['plugins']['entries']['elevenlabs']['enabled'] = True
cfg['plugins']['entries']['elevenlabs']['apiKey'] = 'API_KEY_HERE'
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
ElevenLabs should show as enabled.

## If API key is invalid

The plugin loads but voice generation fails at runtime. User needs to verify key at elevenlabs.io.
