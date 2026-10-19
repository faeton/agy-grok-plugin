---
name: grok-cancel
description: Cancel an active background Grok job in this workspace. Triggered by /grok-cancel or when stopping a running Grok job.
---

# `/grok-cancel` — Cancel a Running Grok Job

Cancel an active background Grok job in this workspace via [grok-companion.mjs](../../scripts/grok-companion.mjs).

## Argument Shape

`[job-id]` (if omitted, cancels the most recent active job for this workspace).

## Steps

1. Resolve the literal absolute path to [grok-companion.mjs](../../scripts/grok-companion.mjs) relative to this `SKILL.md` file's directory (`<plugin_root>/scripts/grok-companion.mjs`).
2. Call `run_command` with `BypassSandbox: true`, `Cwd` set to the workspace root, and `WaitMsBeforeAsync: 8000`:

```bash
node "/abs/path/to/scripts/grok-companion.mjs" cancel '<raw arguments>'
```

## Output Rules

- Present the command output verbatim.
