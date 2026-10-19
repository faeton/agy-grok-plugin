---
name: grok-result
description: Show the stored final output for a finished Grok job. Triggered by /grok-result or when reading the result of a background Grok job.
---

# `/grok-result` — Read Stored Grok Job Output

Print the stored final output for a finished Grok job via [grok-companion.mjs](../../scripts/grok-companion.mjs).

## Argument Shape

`[job-id]` (if omitted, reads the most recent job for this workspace).

## Steps

1. Resolve the literal absolute path to [grok-companion.mjs](../../scripts/grok-companion.mjs) relative to this `SKILL.md` file's directory (`<plugin_root>/scripts/grok-companion.mjs`).
2. Call `run_command` with `BypassSandbox: true`, `Cwd` set to the workspace root, and `WaitMsBeforeAsync: 5000`:

```bash
node /abs/path/to/scripts/grok-companion.mjs result '<raw arguments>'
```

## Output Rules

- Present the full command output to the user verbatim. Do not summarize or condense it. Preserve the answer/review text, severity ordering, file paths, and any error or `stopReason` message exactly as reported.
- **STOP after presenting the output.** This is review/consultation output: do not act on it, fix issues, or apply patches in this turn.
