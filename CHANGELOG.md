# Changelog

All notable changes to this plugin are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.6.2] — 2026-10-19

### Fixed

- Review prompts no longer carry the working-method instructions twice (the port's own "Working budget" line duplicated 0.6.x's `REVIEW_METHOD`).
- Branch reviews batch `git ls-tree` (synced with claude-grok-plugin 0.6.2).

## [0.6.1] — 2026-10-19

Synced with [`claude-grok-plugin` 0.6.1](https://github.com/faeton/claude-grok-plugin/blob/main/CHANGELOG.md).

### Added

- **Profiles** `--profile fast|thrifty|deep` and `GROK_PLUGIN_PROFILE`; default `fast` = `grok-4.7-build-fast` at `low`.
- **Reviews inline the full text of changed files** (150 KB budget, `GROK_PLUGIN_INLINE_BYTES`).

### Changed

- Default model `grok-4.7-build-fast`, effort `low` (was the CLI default model at `medium`, which timed out on real reviews).
- Review and consult prompts end with a working method (batch reads, whole functions, skip unrelated code).
- Lean child environment now comes from the shared `LEAN_ENV`; Claude / Cursor hooks are off too, and project rules / `CLAUDE.md` stay on.
- Wall-clock timeout 15 → 8 minutes.

### Fixed

- Timeouts report the tool-call count and last activity; unfinished runs keep their working notes; a run that stopped before answering fails instead of completing.

## [0.5.1] — 2026-10-19

Initial Google Antigravity (`agy`) release, ported from [`claude-grok-plugin` 0.5.1](https://github.com/faeton/claude-grok-plugin).

### Added

- **Native Antigravity plugin bundle (`plugin.json`).** Ships with `rules/AGENTS.md`, `agents/grok-consult.md`, 8 command skills (`/grok-consult`, `/grok-review`, `/grok-adversarial-review`, `/grok-rescue`, `/grok-status`, `/grok-result`, `/grok-cancel`, `/grok-setup`), 3 internal helper skills (`grok-cli-runtime`, `grok-prompting`, `grok-result-handling`), and `scripts/grok-companion.mjs`.
- **Antigravity outer-sandbox detection (`ensureHostAccess`).** Detects when `grok-companion.mjs` is invoked inside Antigravity's outer terminal sandbox (`BypassSandbox: false`), where Seatbelt/Landlock blocks reading `~/.grok` with `EPERM`/`EACCES`, and fails loudly with a clear `Operation not permitted` retry hint instead of falsely reporting `grok` as uninstalled or logged out.
- **Automatic `~/.grok/bin/grok` and `~/.local/bin/grok` resolution** when `GROK_BIN` is unset and `grok` is not on the default system `PATH`.
- **Auto-approvable command shapes.** Skills and subagent instructions invoke `node <literal-path>/scripts/grok-companion.mjs` without shell variable expansions (`$VAR`) or command substitutions (`$(...)`) so Antigravity's prefix-based command approval works across invocations.
