---
name: code-reading
description: "Support a code or documentation reading session with the user: answer short judgement questions directly, verify every claim against the implementation before writing, propose exact replacement text, confirm changes through the question dialog, and stage edits without committing. Use when the user reads a branch and asks whether an entry, comment, or paragraph is too long, whether to split it, whether wording is accurate, or what a piece of code does, and for CHANGELOG, README, comment, and design-note wording questions."
---

# Code Reading With The User

The user is reading a branch and asks short judgement questions about it. Serve that reading: answer, verify, propose, then change only what was asked.

## Answer

- Give the conclusion first, then the minimum reasoning that supports it. Keep it short.
- Do not expand into design review, refactoring, or background the user did not ask for.
- Treat "is this too long", "should this be split", and "is this accurate" as judgement questions: take a position instead of surveying options.
- Match the user's language.

## Verify Before Writing

- Read the implementation before claiming what it does. Endpoint methods and paths, defaults, status codes, config keys, and feature gating come from the code, not from the sentence under discussion.
- Cite the evidence as `file:line`.
- Judge an entry against the file's own conventions, and read neighbouring entries as the baseline. Length alone is not a defect.
- Disagree when the evidence disagrees; do not confirm the user's framing by default.

## Propose, Then Confirm

- State the exact replacement text the user would paste, not a description of the intended change.
- Split an entry along independently referenceable, user-visible outcomes, not per endpoint or per implementation step; a per-endpoint list is an implementation inventory.
- Reuse the file's existing phrasing for the same concept so terminology stays uniform.
- Confirm through the question dialog rather than making the user send another prompt. Keep the options few and concrete.

## Edit

- Change only what was asked and confirmed; leave neighbouring entries alone.
- Keep each prose paragraph and list item on one source line.
- After editing, `git add` the touched files and stop there. Do not commit, push, rebase, or run any other history operation unless the user explicitly asks for it.
- Report what changed and the evidence that justified it, briefly.
