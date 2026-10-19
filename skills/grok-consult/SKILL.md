---
name: grok-consult
description: Ask xAI's Grok a free-form question, optionally with file context (strictly read-only). Triggered by /grok-consult or when the user asks to consult Grok on a question.
---

# `/grok-consult` — Read-Only Grok Consultation

Ask xAI's Grok a question via [grok-companion.mjs](../../scripts/grok-companion.mjs). **Consultation only** — Grok runs strictly read-only and cannot edit files; do not act on its answer in this turn.

## Argument Shape

`[--wait|--background] [--model <id>] [--effort <low|medium|high|xhigh|max>] [--file <path>]... -- <question>`

- Everything after `--` is the question text.
- `--file <path>` (repeatable) attaches a file as context. Resolve paths relative to the current working directory.
- `--model`, `--effort`, `--reasoning-effort` are passed through to Grok. The companion drops effort flags automatically if the target model does not support them.
- If you cannot tell the question from the flags (no `--` separator and it is ambiguous), use `ask_question` once to clarify.

## Execution Mode

- If the arguments include `--wait`, run in the foreground.
- If they include `--background`, pass `--background` to the companion (the companion detaches its own worker; do NOT use Antigravity's background task mode).
- Otherwise default to foreground.

## Running the Companion

1. Resolve the literal absolute path to [grok-companion.mjs](../../scripts/grok-companion.mjs) relative to this `SKILL.md` file's directory (`<plugin_root>/scripts/grok-companion.mjs`).
2. **Shell quoting (required):** pass all arguments after `consult` as ONE single-quoted shell word. Escape every `'` inside them as `'\''`. Never paste arguments unquoted or inside double quotes (`$(...)`, backticks, `;` and globs in a question would otherwise be evaluated by the shell). Do not use shell variables (`$VAR`) in the command line.
3. Call `run_command` with `BypassSandbox: true`, `Cwd` set to the workspace root, and `WaitMsBeforeAsync: 10000`:

**Foreground:**
```bash
node "/abs/path/to/scripts/grok-companion.mjs" consult '<raw arguments>'
```
If the foreground command is still running after 10 seconds and goes to a background task, wait for the task completion message (do not poll in a loop) and then return its stdout verbatim.

**Background (`--background`):**
```bash
node "/abs/path/to/scripts/grok-companion.mjs" consult '--background <raw arguments>'
```
Then tell the user to check `/grok-status <id>` and `/grok-result <id>`.

## Output Rules

- Return the companion stdout verbatim.
- **STOP after presenting the output.** Do not modify files or apply suggestions unless the user explicitly asks in a follow-up turn.
- If the companion reports a failure (timeout, stall, `stopReason` cancellation, or grok not ready), return that message verbatim — do not substitute your own answer.
