---
name: grok-prompting
description: Internal guidance for composing sharp Grok prompts for consultation and code review inside the Grok plugin.
---

# Grok Prompting

Use this skill when `grok-consult` needs to tighten the user's request into a sharper Grok prompt before making the single companion call. Prompt drafting is the only agent-side work allowed — never solve the task yourself.

## Principles

- **One clear ask per run.** Split unrelated questions into separate consults.
- **State what a good answer looks like** (format, length, what to optimize for) instead of assuming Grok will infer it.
- **Give Grok grounding via `--file <path>`** rather than pasting huge blobs into the question — the companion attaches files cleanly and caps their size.
- **Frame second opinions adversarially:** ask Grok to find what is *wrong*, *fragile*, or *risky*, not merely to agree. Grok is most valuable as an independent skeptic.

## Recipe for a Consult

1. Lead with the concrete question in one or two sentences.
2. Add the decision or constraint that matters ("must stay zero-dependency", "targeting Node 20", "this is hot-path code").
3. Name the output shape: "answer in <=5 bullets", "give the literal flags", "rank the options by failure risk".
4. Attach relevant files with `--file <path>` instead of describing them from memory.

## Recipe for a Review

- Prefer `review` (it builds the diff prompt from local git state automatically).
- Use `adversarial-review <focus>` when the user wants a targeted hunt (e.g. "concurrency bugs in the new worker pool", "auth bypass paths").
- Keep focus text specific and falsifiable.

Keep Grok requests compact. The relay can cancel oversized generations — narrow the scope rather than sending one giant prompt.
