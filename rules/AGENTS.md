# Grok Plugin Rules (Read-Only Consultation & Review)

This plugin integrates xAI's `grok` CLI as a **strictly read-only** second opinion and code reviewer inside Google Antigravity.

## Hard Rules

1. **Never invoke `grok` directly.** Always call the companion script (`scripts/grok-companion.mjs` inside this plugin) so every invocation runs with the read-only tool allowlist (`--tools`), mutating/MCP tool removal (`--disallowed-tools`), capability denials (`--deny Write --deny Edit --deny Bash --deny MCPTool`), `--permission-mode dontAsk`, `--sandbox read-only`, `--no-subagents --no-memory`, and idle/wall-clock watchdogs.
2. **Always run `grok-companion.mjs` with `BypassSandbox: true`.** Antigravity's outer terminal sandbox blocks access to `~/.grok`, blocks network calls to xAI, and prevents macOS `sandbox-exec` from applying Grok's own `read-only` Seatbelt profile. Pass `BypassSandbox: true` on `run_command` when executing `grok-companion.mjs`, and invoke `node <literal-plugin-path>/scripts/grok-companion.mjs ...` directly (without `$VAR` or `$(...)` expansions) so command approvals stay prefix-matchable.
3. **Never auto-fix after consult or review.** When presenting output from `/grok-consult`, `/grok-review`, `/grok-adversarial-review`, `/grok-rescue`, or `/grok-result`, return the companion output verbatim and **STOP**. Ask the user which findings (if any) they want addressed before modifying any file.
4. **Honest failures.** If `grok-companion.mjs` reports a timeout, stall, relay cancellation (`stopReason=cancelled`), or setup issue, report that message verbatim and stop — never substitute your own answer or silently retry.
