---
name: grok-setup
description: Check whether the local Grok CLI is installed, authenticated, and ready for read-only consultations and code reviews. Triggered by /grok-setup.
---

# `/grok-setup` — Verify Local Grok CLI Setup

Check whether the local `grok` CLI is installed, authenticated, and ready via [grok-companion.mjs](../../scripts/grok-companion.mjs).

## Steps

1. Resolve the literal absolute path to [grok-companion.mjs](../../scripts/grok-companion.mjs) relative to this `SKILL.md` file's directory (`<plugin_root>/scripts/grok-companion.mjs`).
2. Call `run_command` with `BypassSandbox: true`, `Cwd` set to the workspace root, and `WaitMsBeforeAsync: 5000`:

```bash
node /abs/path/to/scripts/grok-companion.mjs setup --json
```

## Output Rules

Present the result to the user:
- If `ready` is `true`, say Grok is ready (and show the `grok` version and sandbox status).
- If the `grok` CLI is missing, tell the user to install it and make sure `grok` is on their `PATH` (or in `~/.grok/bin` / `~/.local/bin`).
- If the CLI is present but not authenticated, tell the user to run `grok login` in their terminal.
- Surface the `nextSteps` list verbatim.
- Do not attempt to install or log in on the user's behalf.
