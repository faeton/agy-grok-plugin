---
name: grok-adversarial-review
description: Run a focused/adversarial Grok code review with extra reviewer instructions (strictly read-only). Triggered by /grok-adversarial-review or when the user asks Grok to hunt for specific bug classes in their changes.
---

# `/grok-adversarial-review` — Focused Adversarial Grok Review

Run a Grok code review with extra adversarial focus instructions via [grok-companion.mjs](../../scripts/grok-companion.mjs). Same read-only guarantees as `/grok-review`; do not fix issues in this turn.

## Argument Shape

`[--wait|--background] [--base <ref>] [--scope auto|working-tree|branch] [--model <id>] [--effort <low|medium|high|xhigh|max>] [focus text]`

- Any non-flag text is the focus instruction for the reviewer (e.g. `"hunt for race conditions in the new worker pool"`). It is forwarded as the trailing positional text.
- `--scope`, `--base`, `--model`, `--effort` pass through.
- Execution mode (`--wait` / `--background` / estimate size and call `ask_question` once) follows the exact same rules as [grok-review](../grok-review/SKILL.md).

## Running the Companion

1. Resolve the literal absolute path to [grok-companion.mjs](../../scripts/grok-companion.mjs) relative to this `SKILL.md` file's directory (`<plugin_root>/scripts/grok-companion.mjs`).
2. **Shell quoting (required):** pass the raw arguments to the companion as ONE single-quoted shell word, escaping any `'` inside them as `'\''`. Never paste arguments unquoted or inside double quotes, and do not use shell variables (`$VAR`) in the command line.
3. Call `run_command` with `BypassSandbox: true`, `Cwd` set to the workspace root, and `WaitMsBeforeAsync: 10000`:

**Foreground:**
```bash
node "/abs/path/to/scripts/grok-companion.mjs" adversarial-review '<raw arguments>'
```
If the foreground command is sent to a background task while Grok runs, wait for the task completion notification and then return its stdout verbatim.

**Background (`--background`):**
```bash
node "/abs/path/to/scripts/grok-companion.mjs" adversarial-review '--background <raw arguments>'
```
Then tell the user: `"Grok adversarial review started in the background. Check /grok-status for progress and /grok-result <id> for findings."`

## Output Rules

- Return the companion stdout verbatim.
- **STOP after presenting the review.** Do not fix any issues mentioned until the user explicitly asks.
- If the companion reports a failure, return it verbatim.
