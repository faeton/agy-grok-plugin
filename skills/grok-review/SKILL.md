---
name: grok-review
description: Run a strictly read-only Grok code review against local git state (working tree or branch diff). Triggered by /grok-review or when the user asks Grok to review local changes.
---

# `/grok-review` — Read-Only Grok Code Review

Run a Grok code review through [grok-companion.mjs](../../scripts/grok-companion.mjs). **Review only** — Grok runs strictly read-only and web search is disabled; do not fix issues or apply patches in this turn.

## Argument Shape

`[--wait|--background] [--base <ref>] [--scope auto|working-tree|branch] [--model <id>] [--effort <low|medium|high|xhigh|max>]`

- Preserve the user's arguments (`--scope`, `--base`, `--model`, `--effort` pass through to the companion).
- `/grok-review` does not take focus text. For focused/adversarial framing use `/grok-adversarial-review`.

## Execution Mode Rules

1. If the user's arguments include `--wait`, do not ask. Run in the foreground.
2. If the user's arguments include `--background`, do not ask. Run with the companion's `--background` flag.
3. Otherwise, estimate the review size first (in standard sandbox mode `BypassSandbox: false`):
   - Working tree: `git status --short --untracked-files=all`, `git diff --shortstat --cached`, and `git diff --shortstat`.
   - Base branch: `git diff --shortstat <base>...HEAD`.
   - Treat untracked files as reviewable even when the diff stat is empty.
   - Recommend waiting only when the change is clearly tiny (~1-2 files). Otherwise recommend background.
   - Call `ask_question` once with two options, placing the recommended option first prefixed with `(Recommended)`:
     - `Wait for results`
     - `Run in background`

## Running the Companion

1. Resolve the literal absolute path to [grok-companion.mjs](../../scripts/grok-companion.mjs) relative to this `SKILL.md` file's directory (`<plugin_root>/scripts/grok-companion.mjs`).
2. **Shell quoting (required):** pass the raw arguments to the companion as ONE single-quoted shell word, escaping any `'` inside them as `'\''`. Never paste arguments unquoted or inside double quotes, and do not use shell variables (`$VAR`) in the command line.
3. Call `run_command` with `BypassSandbox: true`, `Cwd` set to the workspace root, and `WaitMsBeforeAsync: 10000`:

**Foreground:**
```bash
node "/abs/path/to/scripts/grok-companion.mjs" review '<raw arguments>'
```
If the foreground command is sent to a background task while Grok runs, wait for the task completion notification and then return its stdout verbatim.

**Background (`--background`):**
```bash
node "/abs/path/to/scripts/grok-companion.mjs" review '--background <raw arguments>'
```
Then tell the user: `"Grok review started in the background. Check /grok-status for progress and /grok-result <id> for findings."`

## Output Rules

- Return the companion stdout verbatim.
- **STOP after presenting the review.** Do not fix any issues mentioned until the user explicitly asks which ones to address.
- If the companion reports a failure, return it verbatim.
