---
name: config-write
description: The only approved method for modifying openclaw.json. Use this Python3 pattern every time — no other method is permitted.
---

# Config Write — Python3 Only

## The only approved pattern

```bash
python3 - <<'PY'
import json

p = '/home/node/.openclaw/openclaw.json'
with open(p) as f:
    cfg = json.load(f)

# --- make your changes to cfg here ---
# example: cfg['channels']['telegram']['token'] = 'YOUR_TOKEN'

with open(p, 'w') as f:
    json.dump(cfg, f, indent=2)
print('done')
PY
```

## Validate after every write

```bash
python3 -c "import json; json.load(open('/home/node/.openclaw/openclaw.json')); print('valid')"
```

If invalid — restore from backup immediately:
```bash
cp /home/node/.openclaw/openclaw.json.bak /home/node/.openclaw/openclaw.json
```

## These are banned — do not use under any circumstances

| Banned method | Why |
|---|---|
| `openclaw config set` | Doesn't support all paths, silently corrupts nested config |
| `node -e` | Token escaping breaks with special characters |
| `jq` | May not be installed, error-prone with nested JSON |
| `sed` / `awk` | Will corrupt JSON formatting |
| File `edit` tool on openclaw.json | Requires exact string match, fragile, wrong tool for JSON |
| `gateway.config.patch` | Not a valid command |

## If Python3 fails

Stop. Do not try any other method. Report the exact Python3 error to the user.
