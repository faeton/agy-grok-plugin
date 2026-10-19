---
name: grok-result-handling
description: Internal guidance for presenting Grok companion output verbatim back to the user without auto-fixing.
---

# Grok Result Handling

When the companion returns Grok's output:

- **Return it verbatim.** Preserve the summary, the severity grouping (`HIGH` / `MEDIUM` / `LOW`) for reviews, and file paths / line numbers exactly as reported.
- **Keep evidence boundaries:** if Grok marked something as an inference, uncertainty, or open question, keep that distinction intact.
- **If there are no findings,** say so plainly and keep any residual-risk note brief.

## Hard Rules

- **CRITICAL: After presenting review or consult output, STOP.** Do not make code changes, fix issues, or apply patches. Grok here is a strictly read-only second opinion. You MUST explicitly ask the user which issues, if any, they want addressed before touching a single file. Auto-applying fixes is forbidden even if a fix looks obvious.
- **Do not turn a failed or incomplete Grok run into your own implementation attempt.** If the companion reports a timeout, a stall, a `stopReason` cancellation, an empty response, or grok-not-ready, report that verbatim and stop — do not substitute your own answer.
- **If the companion reports setup or auth issues,** direct the user to `/grok-setup`. Do not improvise an alternate auth flow.
- **For background jobs,** point the user at `/grok-status <id>` and `/grok-result <id>` rather than guessing at progress.
