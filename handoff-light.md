# Handoff: [short, descriptive title of the change]

<!-- ==================== FORMATTER RULES ====================
You are restructuring a human-written prompt into this template.
DELETE this entire comment block from the finished handoff.

1. PRESERVE VERBATIM every concrete detail from the source prompt:
   names, numbers, copy text, constraints.
2. Pin real file paths from the repo. Mark any path you could not
   verify as [VERIFY PATH].
3. Missing or ambiguous info → [NEEDS CLARIFICATION: <specific
   question>]. Never invent a value or requirement.
4. A section that doesn't apply gets DELETED, not filled.
5. This template is for SINGLE-PHASE changes. If you find yourself
   needing phases, checkpoints, or data/schema changes, say the
   Full template should be used instead — don't force it in here.
========================================================== -->

## Read first

[One line: the doc(s) to read first — e.g. `CLAUDE.md`. This change sits on top of the existing architecture; nothing changes beyond what's listed below.]

## What we're building

[1–3 sentences: old behavior → new behavior. Then the files involved, real paths:]

- `path/to/file` — [what changes here]
- `path/to/other` — [what changes here]

## Spec

- [ ] [Each behavior as a checkable item, precise enough that two implementers would build the same thing]
- [ ] [Edge cases stated explicitly, not left to be inferred]
- [ ] [Undecided values marked [NEEDS CLARIFICATION: ...] — confirm, don't invent]
- [ ] [Intentionally minimal parts marked: "no animation", "no loading state"]

## Out of scope

- No refactoring of surrounding code.
- [Other specific adjacent things NOT to do, even if they'd seem like natural additions.]

## Verification & handback

- Run: [exact commands, verbatim — e.g. `npm test`, `npm run lint` — and the expected results]
- Acceptance:
  - [ ] [command or manual check + expected outcome, one per behavior above]
- If a documented assumption turns out wrong against the real code, stop and flag it — don't guess and continue.
- Handback: files touched, the verification output as evidence (never assert success without it), a suggested commit message (the user commits manually), and anything that deviated from this doc.
