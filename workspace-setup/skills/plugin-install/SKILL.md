---
name: plugin-install
description: How to install OpenClaw plugins. Covers bundled plugins and ClawHub/npm plugins with mandatory version matching.
---

# Plugin Install

## Bundled plugins — no npm, no network, install directly

```bash
openclaw plugins install telegram
openclaw plugins install discord
openclaw plugins install duckduckgo
openclaw plugins install device-pair
openclaw plugins install elevenlabs
openclaw plugins install azure-speech
openclaw plugins install browser
openclaw plugins install active-memory
openclaw plugins install memory-core
openclaw plugins install openrouter
openclaw plugins install copilot-proxy
```

## ClawHub/npm plugins — version MUST match the app

These require npm and the version must exactly match the running OpenClaw app version:

```bash
# 1. Get the app version first — always
openclaw --version
# Example: 2026.5.6

# 2. Install with exact version and --force
openclaw plugins install @openclaw/whatsapp@2026.5.6 --force
```

Available ClawHub plugins: `@openclaw/whatsapp`, `@openclaw/slack`, `@openclaw/msteams`, `@openclaw/googlechat`, `@openclaw/line`, `@openclaw/nostr`

**Version mismatch causes `text-utility-runtime` crash on startup. Always use `--force` with the exact version.**

## Check before installing

```bash
openclaw plugins list
```

If the plugin is already listed at the correct version, skip the install.

## After any plugin install — restart required

```bash
kill $(pgrep -f 'node.*dist/index.js') 2>/dev/null; true
```

Wait 20–30 seconds, then verify:
```bash
openclaw plugins list
openclaw status --deep
```
