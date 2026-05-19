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

**Step 2 — Write config (both plugin AND channel entries required)**

Both entries are required. Missing `channels.whatsapp` means WhatsApp loads as a plugin but never connects as a channel. Use `openclaw config patch` — do NOT edit openclaw.json directly:

```bash
echo '{plugins: {entries: {whatsapp: {enabled: true}}}, channels: {whatsapp: {}}}' | openclaw config patch --stdin
```

Expected output: `Applied 1 config update(s). Restart the gateway to apply.`

**Step 3 — Warn user, then restart gateway**

BEFORE running the kill command, send the user this message exactly:
> "Restarting the gateway now to load the WhatsApp plugin — it will go offline for ~30 seconds. Send me any message after it comes back and I'll continue with the QR code step."

Then run:
```bash
kill $(pgrep -f 'node.*dist/index.js') 2>/dev/null; true
```
Wait 20–30 seconds. Verify WhatsApp appears as a channel:
```bash
openclaw status --deep
```
WhatsApp must appear as `UNLINKED` in the channel list. If it does not appear at all, the config patch in Step 2 failed — do not proceed to QR.

**Step 4 — Capture QR code via script**

There is NO `whatsapp_login` tool. There is no `--qr-output` flag. Running `openclaw channels login --channel whatsapp` directly will not work — it requires an interactive TTY.

Run the `script` capture in the background:
```bash
rm -f /tmp/wa_login.typescript
script -q -f -c 'openclaw channels login --channel whatsapp' /tmp/wa_login.typescript &
```

Wait 8 seconds, then proceed to Step 5.

**Step 5 — Convert ANSI capture to PNG and deliver QR**

YOU MUST run this Python script. Do not skip it. Do not send the MEDIA line without running this first:

```bash
python3 - <<'PY'
import re, zlib, struct, time
from pathlib import Path

capture = Path('/tmp/wa_login.typescript')
out = Path('/home/node/.openclaw/media/whatsapp-login-qr.png')
out.parent.mkdir(parents=True, exist_ok=True)

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
out.write_bytes(png)
print('QR saved to', out)
PY
```

If the script prints `QR saved to ...`, send this message exactly:
```
Your QR code is ready!

MEDIA:/home/node/.openclaw/media/whatsapp-login-qr.png

If you don't see the image, refresh the page — it will appear after reload.

Once you can see it: WhatsApp → Settings → Linked Devices → Link a Device → scan the QR.
```

If the script raises `QR not ready`, wait 5 more seconds and run it again.

**Step 6 — User scans QR — then kill capture and ask for their number**

Once the user confirms they scanned it:

1. Kill the background capture process immediately — it competes with the plugin's Baileys connection:
```bash
pkill -f 'openclaw channels login' 2>/dev/null; pkill -f 'script.*wa_login' 2>/dev/null; true
```

2. Ask the user: "What is your WhatsApp phone number including country code? (e.g. +15551234567)"

**Step 7 — Write allowFrom file with user's number**

Replace `+1XXXXXXXXXX` with the number they provided:
```bash
mkdir -p /home/node/.openclaw/credentials
echo '{"version":1,"allowFrom":["+1XXXXXXXXXX"]}' > /home/node/.openclaw/credentials/whatsapp-default-allowFrom.json
```

Without this file, all inbound messages are silently dropped regardless of pairing status.

**Step 8 — Restart gateway for clean Baileys reconnect**

The Baileys WebSocket goes stale after the QR pairing sequence. A restart is required for it to start receiving messages:

Send the user: "Restarting gateway one more time to activate message receiving — ~30 seconds, then send me a test WhatsApp message."

```bash
kill $(pgrep -f 'node.*dist/index.js') 2>/dev/null; true
```

**Step 9 — Verify and confirm end-to-end**

After gateway restarts, run:
```bash
openclaw status --deep
```
Should show `WhatsApp │ LINKED │ linked`.

Then ask the user to send a test WhatsApp message to the number. Confirm it is received and the main AI responds. Setup is complete only when the user confirms they got a response.

## If plugin install fails

Stop. Check version match. Do not try bundled path or alternatives unless the runbook explicitly says to.

## If QR expires or script prints "QR not ready"

```bash
kill %1 2>/dev/null; true
rm -f /tmp/wa_login.typescript
script -q -f -c 'openclaw channels login --channel whatsapp' /tmp/wa_login.typescript &
```
Wait 8 seconds, then re-run the Python converter from Step 5.
