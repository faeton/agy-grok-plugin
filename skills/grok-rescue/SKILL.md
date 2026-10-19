---
name: grok-rescue
description: Hand a question or code review to xAI's Grok via the grok-consult subagent. Triggered by /grok-rescue or when delegating a consultation/review to the grok-consult subagent.
---

# `/grok-rescue` — Delegate to the `grok-consult` Subagent

Forward the user's request to the `grok-consult` custom subagent (xAI Grok).

## Argument Shape

`[--wait|--background] [--file <path>]... <question or review request>`

## Routing Rules

1. Invoke the `grok-consult` subagent via the `invoke_subagent` tool:
   - `TypeName`: `"grok-consult"`
   - `Role`: `"Grok Consultant"`
   - `Model`: `"inherit"`
   - `Workspace`: `"inherit"`
   - `Prompt`: Forward the user's request (plus the literal absolute path to [grok-companion.mjs](../../scripts/grok-companion.mjs) resolved relative to this `SKILL.md` file so the subagent can run it immediately in one step).
2. Do not answer the question or perform the review yourself — the subagent forwards it to Grok.
3. `--file <path>` and any `--effort` / `--model` flags are for Grok — leave them in the forwarded text so the subagent passes them through (strip `--wait` / `--background` from the forwarded text, since the subagent itself runs asynchronously via `invoke_subagent`).
4. When the subagent replies, return its output verbatim. Do not paraphrase, summarize, or act on Grok's findings in this turn.
