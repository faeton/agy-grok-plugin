---
name: grok-status
description: Show active and recent Grok background jobs for this workspace. Triggered by /grok-status or when checking on Grok job progress.
---

# `/grok-status` — Inspect Grok Jobs

Show active and recent Grok jobs for this workspace via [grok-companion.mjs](../../scripts/grok-companion.mjs).

## Argument Shape

`[job-id] [--wait] [--timeout-ms <ms>] [--all]`

## Steps

1. Resolve the literal absolute path to [grok-companion.mjs](../../scripts/grok-companion.mjs) relative to this `SKILL.md` file's directory (`<plugin_root>/scripts/grok-companion.mjs`).
2. Call `run_command` with `BypassSandbox: true`, `Cwd` set to the workspace root, and `WaitMsBeforeAsync: 10000`:

```bash
node /abs/path/to/scripts/grok-companion.mjs status '<raw arguments>'
```

## Output Rules

- If no job ID was passed:
  - Render the output as a single compact Markdown table of current and recent jobs.
  - Preserve the `Job`, `Kind`, `Status`, `Phase`, `Elapsed`, and `Summary` columns. No extra prose.
- If a job ID was passed:
  - Present the full command output verbatim. Do not summarize.
  - If the job is complete, remind the user they can run `/grok-result <id>` for the full output.
