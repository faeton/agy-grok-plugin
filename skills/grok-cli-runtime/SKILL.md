---
name: grok-cli-runtime
description: Internal helper contract for calling the grok-companion runtime from Google Antigravity. Use inside the grok-consult subagent and when executing Grok consultation or review commands.
---

# Grok Runtime (`grok-companion.mjs`)

Use this skill inside the `grok-consult` subagent and when executing Grok slash commands (`/grok-consult`, `/grok-review`, `/grok-adversarial-review`, `/grok-status`, `/grok-result`, `/grok-cancel`, `/grok-setup`).

## Locating the Companion Script

The companion script lives at [grok-companion.mjs](../../scripts/grok-companion.mjs) relative to this `SKILL.md` file (i.e. `<plugin_root>/scripts/grok-companion.mjs`, typically `/Users/<user>/.gemini/antigravity-cli/plugins/grok/scripts/grok-companion.mjs` when installed via `agy plugin install`).

Always resolve the **literal absolute path** to `grok-companion.mjs` from the path of the `SKILL.md` file you viewed, and invoke `node` directly without `$VAR` or `$(...)` shell expansions so Antigravity command approvals remain prefix-matchable.

## Primary Helpers (One `run_command` Call Each, Never Chained)

- `node "/abs/path/to/scripts/grok-companion.mjs" consult '[flags] -- <question>'`
- `node "/abs/path/to/scripts/grok-companion.mjs" review '[flags]'`
- `node "/abs/path/to/scripts/grok-companion.mjs" adversarial-review '[flags] [focus text]'`

## Execution Rules

1. **Always pass `BypassSandbox: true` on `run_command`.**
   - `grok-companion.mjs` reads `~/.grok/auth.json` and `~/.grok/models_cache.json`, writes job state to `~/.grok/companion/`, calls xAI's network API, and spawns `grok` under Grok's own OS `read-only` sandbox (`sandbox-exec` on macOS, `bwrap`/Landlock on Linux).
   - Antigravity's outer terminal sandbox blocks `~/.grok`, blocks network access, and prevents nested `sandbox-exec` on macOS.
2. **Thin forwarder, not an orchestrator.** Invoke `grok-companion.mjs` once and return stdout verbatim.
3. **Strictly read-only.** The companion allowlists Grok's read-only tools (`read_file`, `grep`, `list_dir`, plus `web_search`, `web_fetch` for consult), removes mutating and MCP tools, denies `Write`/`Edit`/`Bash`/`MCPTool`, sets `--permission-mode dontAsk`, runs Grok under `--sandbox read-only`, disables subagents and memory, and supervises execution with idle and wall-clock watchdogs. Never call `grok` directly.
4. **Effort & Model.** Leave `--effort` and `--model` unset unless the user explicitly requested them. Effort defaults to `medium` (overridable via `GROK_PLUGIN_EFFORT`); the companion drops it automatically if the target model does not support reasoning effort.
5. **Shell quoting.** Pass the raw argument string to the companion as ONE single-quoted shell word, escaping any embedded `'` as `'\''`. Never pass user text unquoted or inside double quotes.
6. **Foreground inside `grok-consult` subagent.** Inside the `grok-consult` subagent, always run in the foreground — never pass `--background` (strip `--wait` and `--background` from forwarded text). Subagents already run asynchronously via `invoke_subagent`; detaching a background worker inside a subagent orphans the worker when the subagent finishes.

## Command Selection

- Free-form question or "what do you think of X" → `consult`. Put the whole question after `--`.
- "Review my changes / this diff / this branch" → `review` (add `--scope` / `--base` if specified).
- "Review, and specifically hunt for `<X>`" → `adversarial-review <X>`.
- Background jobs (created by `/grok-consult --background` or `/grok-review --background` in the main thread) are inspected via `/grok-status`, `/grok-result <id>`, and `/grok-cancel <id>`.

## Failure Handling

- If the companion reports `grok` is missing or unauthenticated, return that verbatim and stop (point the user to `/grok-setup`).
- If it reports a timeout, stall, or `stopReason` cancellation, return that verbatim. Do not retry silently or substitute your own answer.
