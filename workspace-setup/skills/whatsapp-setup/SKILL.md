---
name: whatsapp-setup
description: Step-by-step procedure for setting up WhatsApp on an OpenClaw instance. Trigger on any request to connect, set up, or configure WhatsApp.
---

# WhatsApp Setup

Read the full runbook before starting:
```
https://raw.githubusercontent.com/almnjoy/madeformeai-private-runbooks/main/WHATS-APP-SETUP.md
```

## Steps (summary — runbook is authoritative)

**Step 1 — Get app version, install matching plugin**
```bash
openclaw --version
# Use that exact version below
openclaw plugins install @openclaw/whatsapp@<VERSION> --force
```
Version mismatch = `text-utility-runtime` crash. Always use `--force` with exact version.

**Step 2 — Warn user, then restart gateway**

BEFORE running the kill command, send the user this message exactly:
> "Restarting the gateway now to load the WhatsApp plugin — it will go offline for ~30 seconds. Send me any message after it comes back and I'll continue with the QR code step."

Then run:
```bash
kill $(pgrep -f 'node.*dist/index.js') 2>/dev/null; true
```
Wait 20–30 seconds. Verify plugin loaded:
```bash
openclaw plugins list
openclaw status --deep
```

**Step 3 — Capture QR code via script**

There is NO `whatsapp_login` tool. There is no `--qr-output` flag. Running `openclaw channels login --channel whatsapp` directly will not work because it requires an interactive TTY.

Run the `script` capture in the background:
```bash
rm -f /tmp/wa_login.typescript
script -q -f -c 'openclaw channels login --channel whatsapp' /tmp/wa_login.typescript &
```

Wait 8 seconds, then proceed immediately to Step 4.

**Step 4 — Convert ANSI capture to PNG and output as inline image**

YOU MUST run this Python script. Do not skip it. The script prints a base64 data URI — copy the entire output and paste it into your chat response so the image renders inline.

```bash
python3 - <<'PY'
import re, zlib, struct, time, base64
from pathlib import Path

capture = Path('/tmp/wa_login.typescript')

for _ in range(20):
    raw = capture.read_text(errors='ignore') if capture.exists() else ''
    lines = []
    for line in raw.splitlines():
        m = re.search('\x1b\\[47m\x1b\\[30m(.*?)\x1b\\[0m', line)
        if m:
            c = m.group(1)
            if len(c) > 20 and not set(c) <= {'▀'}:
                lines.append(c)
    if len(lines) >= 31:
        break
    time.sleep(1)

if len(lines) < 31:
    raise SystemExit(f'QR not ready — got {len(lines)} lines. Wait and retry.')

lines = lines[:31]
w = max(map(len, lines))
lines = [l.ljust(w) for l in lines]

modules = []
for l in lines:
    top, bot = [], []
    for ch in l:
        if ch == ' ':   top.append(0); bot.append(0)
        elif ch == '█': top.append(1); bot.append(1)
        elif ch == '▀': top.append(1); bot.append(0)
        elif ch == '▄': top.append(0); bot.append(1)
        else:           top.append(0); bot.append(0)
    modules.extend([top, bot])

scale = 8; border = 32
width  = w * scale + 2 * border
height = len(modules) * scale + 2 * border

rows = []
for y in range(height):
    row = bytearray([0])
    my = (y - border) // scale
    for x in range(width):
        mx = (x - border) // scale
        black = 0 <= my < len(modules) and 0 <= mx < w and modules[my][mx]
        row += b'\x00\x00\x00' if black else b'\xff\xff\xff'
    rows.append(bytes(row))

def chunk(t, d):
    return struct.pack('>I', len(d)) + t + d + struct.pack('>I', zlib.crc32(t+d) & 0xffffffff)

png = (
    b'\x89PNG\r\n\x1a\n'
    + chunk(b'IHDR', struct.pack('>IIBBBBB', width, height, 8, 2, 0, 0, 0))
    + chunk(b'IDAT', zlib.compress(b''.join(rows), 9))
    + chunk(b'IEND', b'')
)
b64 = base64.b64encode(png).decode()
print(f'![WhatsApp QR](data:image/png;base64,{b64})')
PY
```

The script prints one line starting with `![WhatsApp QR](data:image/png;base64,...)`

Send that entire line as your chat message, followed by:
```
Scan this QR in WhatsApp: Settings → Linked Devices → Link a Device → scan.
```

Do NOT send `MEDIA:` paths — they do not render. The data URI above is the only method that works.

If the script raises `QR not ready`, wait 5 more seconds and run it again.

**Step 5 — User scans QR**
Wait for the user to confirm they scanned it.

**Step 6 — Verify**
```bash
openclaw status --deep
```
Success:
```
WhatsApp │ ON │ OK │ configured · accounts 1/1
WhatsApp │ LINKED │ linked
```

**Step 7 — Approve sender (if needed)**
```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

## If plugin install fails

Stop. Check version match. Do not try bundled path or alternatives unless the runbook explicitly says to.

## If QR expires or script prints "QR not ready"

```bash
kill %1 2>/dev/null; true
rm -f /tmp/wa_login.typescript
script -q -f -c 'openclaw channels login --channel whatsapp' /tmp/wa_login.typescript &
```
Wait 8 seconds, then re-run the Python converter from Step 4 and output the new data URI.
