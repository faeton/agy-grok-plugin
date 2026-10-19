# Grok plugin for Google Antigravity (`agy`)

Consult [xAI's Grok CLI](https://x.ai) from inside Google Antigravity (`agy` CLI, Antigravity App, and Antigravity IDE) for a **strictly read-only** second opinion or code review — without ever letting Grok touch your files.

Think of it as the Antigravity counterpart to [`claude-grok-plugin`](https://github.com/faeton/claude-grok-plugin) and OpenAI's [`codex-plugin-cc`](https://github.com/openai/codex-plugin-cc): you stay in Antigravity, and hand specific questions or diffs to Grok when you want an independent perspective. Grok answers; you decide what to do with it.

```text
/grok-consult -- is a sync.Map the right choice for this hot-path cache?
/grok-review --scope working-tree
/grok-adversarial-review hunt for race conditions in the new worker pool
```

---

## Why this exists

Grok's CLI is a full agent — by default it can edit files, run shell commands, and act on your repo. That is the opposite of what you want from a *consultation*: when you ask for a second opinion you want an answer, not an agent quietly rewriting your code. Early naive wrappers had three problems:

1. **Grok edited files during "consult".** It inherited permissive permission rules and happily wrote to the repo.
2. **It hung or silently failed.** A relay-cancelled request returned empty stdout with exit code 0 — indistinguishable from success.
3. **No job tracking.** Background runs were fire-and-forget with no status.

This plugin fixes all three:

- **Strictly read-only, in layers.** Every Grok invocation *allowlists* only read-only tools (`read_file`, `grep`, `list_dir`, plus web search/fetch for consult), *removes* the mutating tools and the MCP meta-tools, *denies* writes, shell and MCP calls at the permission layer, refuses anything that would need approval (`--permission-mode dontAsk`), and runs Grok inside the OS **`read-only` sandbox** (Seatbelt on macOS, Landlock on Linux) — which also covers anything Grok spawns, such as hooks or local MCP servers. If the sandbox can't be applied, Grok would carry on unsandboxed, so the companion stops the run instead. On Linux the sandbox also needs bubblewrap (`bwrap`) on `PATH`. Without it, runs go ahead without the OS sandbox and show a warning (the tool and permission layers still apply). Set `GROK_COMPANION_REQUIRE_SANDBOX=1` to fail instead. Memory is disabled, and reviews also disable web access.

  The tool lists use two different vocabularies and both matter: `--tools` / `--disallowed-tools` take Grok's **internal tool ids**, while `--allow`/`--deny` take **capability names**. Unknown names are silently ignored — which is why the allowlist exists: the old denylist missed Grok's `write` tool entirely.
- **Outer-sandbox aware.** Antigravity runs shell commands inside its own terminal sandbox by default (`BypassSandbox: false`), which blocks access to `~/.grok`, outbound network requests to xAI, and nested `sandbox-exec` on macOS. This plugin instructs Antigravity to run `grok-companion.mjs` with `BypassSandbox: true` (so Grok can apply its own read-only OS sandbox), and `grok-companion.mjs` actively detects `EPERM`/`EACCES` on `~/.grok` so an accidental sandboxed call fails loudly with a clear retry message instead of falsely reporting `grok` as uninstalled or logged out.
- **Honest failures — in both directions.** A relay cancellation, a stall, or an empty answer is reported as a real, non-zero failure, never as a silent empty success. Just as importantly, a *completed* answer is never thrown away: stop-reason matching is case- and separator-insensitive and fails open on unrecognized values that carry text, because Grok documents its stop-reason list as non-exhaustive. Truncated answers (`max_tokens`, `max_turn_requests`) are returned with a warning rather than discarded.
- **Stall detection.** Runs stream their events, so a stalled relay is caught as an absence of output within minutes instead of burning the full watchdog. A wall-clock backstop still bounds the worst case. Every way a run can end early — timeout, stall, `/grok-cancel`, Ctrl-C — sends SIGTERM to Grok's process group and escalates to SIGKILL before the companion exits, so no process is left behind.
- **Background jobs with status.** Long consultations/reviews run as detached workers tracked in an on-disk registry. Check them with `/grok-status`, read them with `/grok-result`, stop them with `/grok-cancel`. Job state lives outside your repo (`~/.grok/companion/`), so it never dirties the working tree.

---

## Install

### From your shell (`agy plugin`)

```bash
# Install from GitHub
agy plugin install https://github.com/faeton/agy-grok-plugin

# Or install from a local checkout
agy plugin install /path/to/agy-grok-plugin
```

### Inside an interactive `agy` session

```text
/plugin install /path/to/agy-grok-plugin
```

Or register it in `.agents/plugins.json` (workspace) or `~/.gemini/config/plugins.json` (global):

```json
{
  "entries": [
    { "path": "~/Sites" , "include_only": ["agy-grok-plugin"] }
  ]
}
```

Then verify your setup:

```text
/grok-setup
```

### Prerequisites

- Google Antigravity (`agy` CLI, Antigravity App, or Antigravity IDE)
- The `grok` CLI on your `PATH` (or `~/.grok/bin/grok`), authenticated (`grok login`)
- Node.js 18+

`/grok-setup` verifies the CLI is installed and logged in and tells you what to fix if not.

---

## Slash Commands & Skills

In Antigravity, every bundled skill is available both as a **slash command** and via semantic skill discovery:

| Slash Command | What it does |
|---|---|
| `/grok-consult -- <question>` | Ask Grok a free-form question. `--file <path>` (repeatable) attaches context. |
| `/grok-review` | Review your local git changes (working tree or branch diff). |
| `/grok-adversarial-review <focus>` | Review with extra focus instructions (e.g. "hunt for auth bypasses"). |
| `/grok-rescue <request>` | Hand a question/review to the `grok-consult` custom subagent (`invoke_subagent`). |
| `/grok-status [job-id]` | List recent jobs, or show one job (add `--wait` to block until done). |
| `/grok-result [job-id]` | Print the stored final output of a finished job. |
| `/grok-cancel [job-id]` | Cancel a running background job (kills Grok cleanly). |
| `/grok-setup` | Check that the `grok` CLI is installed and authenticated. |

### Common flags

- `--background` — run as a tracked detached job; read it back with `/grok-status` / `/grok-result`. (Foreground is the default for small asks.)
- `--model <id>` — pin a specific Grok model.
- `--effort <low|medium|high|xhigh|max>` — reasoning effort, on `consult`, `review` and `adversarial-review`. **Defaults to `medium`** (the grok CLI's own default is `high`, which is very slow for routine asks). Automatically dropped if the target model doesn't support it (so it never 400s).
- `--scope auto|working-tree|branch` and `--base <ref>` — for reviews.
- `--timeout-ms <ms>` — wall-clock backstop (default 15 minutes; must be > 0).
- `--idle-ms <ms>` — kill a run that produces no output at all for this long (default 3 minutes). Catches a stalled relay long before the backstop. Don't set it much lower: Grok can spend up to 120s draining usage after the answer without emitting anything.

### Changing the default effort

Per call, pass `--effort`, e.g. `/grok-review --effort high`.

To change the default for every call, export `GROK_PLUGIN_EFFORT` in your shell profile (`~/.zshrc`):

```bash
export GROK_PLUGIN_EFFORT="high"
```

Use `"default"` to send no effort flag and let the `grok` CLI decide.

### Examples

```text
# Quick second opinion
/grok-consult -- what are the failure modes of this retry loop?

# Consult with file context
/grok-consult --file src/cache.ts --file src/cache.test.ts -- is the eviction correct?

# Review the current working tree, wait for the result
/grok-review --wait

# Review a branch against main, in the background
/grok-review --scope branch --base origin/main --background
/grok-status            # find the job id
/grok-result <job-id>   # read the findings

# Targeted adversarial review
/grok-adversarial-review concurrency safety of the new connection pool
```

---

## How it works

```text
plugin.json       Antigravity plugin manifest
assets/logo.svg   Plugin card icon
rules/AGENTS.md   Always-on safety rules (read-only via companion, never auto-fix)
agents/           grok-consult custom subagent (thin forwarder via invoke_subagent)
skills/
  grok-consult/             /grok-consult slash command & skill
  grok-review/              /grok-review slash command & skill
  grok-adversarial-review/  /grok-adversarial-review slash command & skill
  grok-rescue/              /grok-rescue slash command & skill
  grok-status/              /grok-status slash command & skill
  grok-result/              /grok-result slash command & skill
  grok-cancel/              /grok-cancel slash command & skill
  grok-setup/               /grok-setup slash command & skill
  grok-cli-runtime/         internal contract: how to invoke the companion
  grok-prompting/           internal contract: how to compose a sharp Grok prompt
  grok-result-handling/     internal contract: how to present output / never auto-fix
scripts/
  grok-companion.mjs        dispatcher: setup/consult/review/adversarial-review/
                            status/result/cancel (+ internal consult-worker)
  lib/
    args.mjs      schema-driven arg parser (handles quotes, escapes, "--" tail)
    grok.mjs      the grok CLI runner: read-only flags, outer-sandbox check,
                  idle + wall-clock watchdogs, streaming event parsing,
                  stop-reason classification
    git.mjs       review-context construction from git state
    state.mjs     on-disk job registry (atomic writes, locked index)
    jobs.mjs      job lifecycle: create → run → terminal; orphan reconciliation
    render.mjs    human-readable output
```

Validate the plugin manifest and components with:

```bash
agy plugin validate . --strict
```

Run the test suite with:

```bash
node scripts/test/runner.test.mjs && node scripts/test/state.test.mjs
```

---

## License

MIT © faeton
