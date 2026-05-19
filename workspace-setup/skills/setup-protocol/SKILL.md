---
name: setup-protocol
description: Core behavioral rules for the Setup Assistant. Apply these rules on every setup request, before taking any action.
---

# Setup Protocol — Rules That Cannot Be Broken

## STEP 0 — Read the runbook before anything else

Every setup task has a runbook. Fetch it FIRST with web_fetch before running a single command:

```
https://raw.githubusercontent.com/almnjoy/madeformeai-private-runbooks/main/<FILENAME>
```

Runbook filenames are listed in TOOLS.md. Do not start setup until you have read the runbook for this specific task.

## Follow steps in exact order

- Execute runbook steps one at a time, in order.
- Do not skip steps.
- Do not reorder steps.
- Do not add steps that are not in the runbook.

## FAILURE RULE — Stop, never improvise

If a step fails:
1. Retry the EXACT same command once.
2. If it fails again — STOP.
3. Report the exact error output to the user.
4. Do NOT try a different command, tool, approach, or workaround.

**No alternatives are acceptable.** The runbook step either works or it doesn't. If it doesn't, stop and report.

## Gateway restart rule

Only restart the gateway when the runbook explicitly says to at that point in the steps.

Always use the kill method — never `openclaw gateway restart`:
```bash
kill $(pgrep -f 'node.*dist/index.js') 2>/dev/null; true
```

Wait 20–30 seconds after killing. K8s recreates the process automatically.

## Verify before moving on

After each major step, run the verification the runbook specifies before proceeding to the next step.

## Never use these — ever

- `openclaw config set` — broken for most config paths
- `node -e` — escaping failures with tokens and special chars
- `jq`, `sed`, `awk` to edit JSON files
- The file `edit` tool on openclaw.json
- `gateway.config.patch`
- `openclaw gateway restart`
- Any npm plugin install without first checking `openclaw --version`
