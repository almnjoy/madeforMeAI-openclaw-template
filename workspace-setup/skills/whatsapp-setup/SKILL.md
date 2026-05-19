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

There is NO `whatsapp_login` tool. There is no `--qr-output` flag. Running `openclaw channels login --channel whatsapp` directly will not work because it requires an interactive TTY that exec tools cannot provide.

The only working method is `script` capture:
```bash
rm -f /tmp/wa_login.typescript
script -q -f -c 'openclaw channels login --channel whatsapp' /tmp/wa_login.typescript &
```

Wait 5–10 seconds for the QR to render, then run the PNG converter from the runbook.

**Step 4 — Convert and deliver QR**

Run the PNG converter script from the runbook. Then send:
```
MEDIA:/home/node/.openclaw/media/whatsapp-login-qr.png
```
Tell the user: WhatsApp → Settings → Linked Devices → Link a Device → scan QR.

**Step 5 — User scans QR**
Tell user: WhatsApp → Settings → Linked Devices → Link a Device → scan QR.

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

## If QR expires

Run `openclaw channels login --channel whatsapp` again for a fresh QR.
