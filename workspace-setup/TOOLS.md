# TOOLS.md - Setup Environment

## Source of Truth Order

Always use sources in this exact order:

1. **Private runbooks** — fetch first, every time, before anything else.
2. **OpenClaw official docs** — for OpenClaw-specific behavior the runbook doesn't cover.
3. **MadeForMeAI public docs** — for user-facing product behavior.
4. Runtime status/config/tools for the current instance.

---

## Private Runbooks Repository

The runbooks repo is **public**. Use `web_fetch` on the raw URL — no token needed.

```text
Repo: https://github.com/almnjoy/madeformeai-private-runbooks
```

### How to fetch a runbook

**Primary method — web_fetch (preferred):**

```
https://raw.githubusercontent.com/almnjoy/madeformeai-private-runbooks/main/<EXACT_FILENAME>
```

Example — WhatsApp runbook:
```
https://raw.githubusercontent.com/almnjoy/madeformeai-private-runbooks/main/WHATS-APP-SETUP.md
```

**Fallback — curl with token (if web_fetch fails):**

```bash
curl -s -H "Authorization: Bearer $RUNBOOKS_READ_GITHUB_TOKEN" \
  https://raw.githubusercontent.com/almnjoy/madeformeai-private-runbooks/main/<EXACT_FILENAME>
```

The env var `$RUNBOOKS_READ_GITHUB_TOKEN` is available in your bash environment. Do not hardcode or print its value.

### Current runbooks — exact filenames (case-sensitive)

```text
telegram-setup.md
discordsetup.md
elevenlabs-voice.md
browser-extension-setup.md
gmail-setup.md
google-drive-setup.md
imessage-setup.md
tailscale-boot-fix.md
WHATS-APP-SETUP.md
job-agent-setup.md
```

### Token rules

- Use read-only access only.
- Never print the token value in chat.
- Never save the token to files, memory, or logs.
- If both web_fetch and curl fail, say why — do not silently skip or proceed without the runbook.

---

## Public Documentation

```text
https://docs.madeformeai.com
https://docs.openclaw.ai/
```

Use only when no runbook covers the topic.

---

## User Environment Reality

Users:
- do not have CLI access
- may not understand terminals, JSON, or server concepts
- want setup to work, not a lecture

Therefore:
- never give terminal commands to the user
- use your own bash/exec tools for all backend steps
- only ask the user to take browser/device/account actions when required by the runbook

---

## Chat/Connector Target Rule

For channel or chat-scoped connectors, ask which chat/channel/account to configure before finalizing setup.

Examples:
- "Which Discord server and channel?"
- "Which WhatsApp number?"

If the runbook defines a clear default and the user has one obvious choice, use it and mention it briefly.

---

## Permission Rule

The user's setup request is permission to proceed. Do not ask "may I do X?" for routine runbook steps.

Ask only when:
- the runbook requires a user choice
- a credential, token, or account approval is needed
- the action is destructive or irreversible

---

## Plugin Management

### Two classes of plugins

**Class 1 — Bundled stock** (in the image, install without npm):

```text
telegram, discord, duckduckgo, device-pair, signal, matrix, mattermost,
irc, imessage, elevenlabs, azure-speech, browser, perplexity, exa,
tavily, firecrawl, searxng, active-memory, memory-core, openrouter, copilot-proxy
```

**Class 2 — ClawHub/npm packages** (require npm):

```text
@openclaw/whatsapp    @openclaw/googlechat    @openclaw/msteams
@openclaw/slack       @openclaw/line          @openclaw/nostr
```

### Install procedure (use your bash tool — never ask the user)

```bash
# Check what's installed
openclaw plugins list

# Install a bundled plugin
openclaw plugins install telegram

# Install a ClawHub plugin
openclaw plugins install @openclaw/whatsapp

# Restart gateway to load new plugins
kill $(pgrep -f 'node.*dist/index.js') || openclaw gateway restart

# Confirm it loaded
openclaw plugins list
```

Installing plugins does not require user permission. It is non-destructive.

---

## Goal

Runbooks → Tools → Working Setup
