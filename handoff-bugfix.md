# Bugfix handoff: [one-line description of the bug]

<!-- ==================== FORMATTER RULES ====================
You are restructuring a human-written bug report into this template.
DELETE this entire comment block from the finished handoff.

1. PRESERVE VERBATIM exact error text, stack traces, repro steps,
   and version numbers from the source prompt.
2. Pin real file paths from the repo. Mark any path you could not
   verify as [VERIFY PATH].
3. Missing or ambiguous info → [NEEDS CLARIFICATION: <specific
   question>]. Never invent details.
4. A section that doesn't apply gets DELETED, not filled.
========================================================== -->

## Read first

[One line: docs to read first — e.g. `CLAUDE.md`. This is a fix, not a refactor — everything not implicated in the bug stays exactly as-is.]

## Symptom

[What actually happens, in plain language. Exact error messages / stack traces verbatim:]

```
[paste error output here]
```

[When it started or what changed around then, if known.]

## Reproduction

1. [Step]
2. [Step]
3. **Expected:** [...] **Actual:** [...]

[If not reliably reproducible, say so — frequency, conditions, environment.]

## Where to look

- Suspected: `path/to/file` — [why]
- Already ruled out: [what, and how — DELETE if nothing ruled out]
- Prior attempts: [what was tried and why it failed — DELETE if none]

## Fix protocol

1. **Diagnose first.** Find the root cause before changing anything. If the root cause turns out to be somewhere other than suspected, or the proper fix requires structural change, stop and flag it before proceeding.
2. **Reproduce before fixing.** Write a failing test that captures the bug. [If no test infrastructure covers this area: reproduce manually and record the exact failing command/steps instead.]
3. **Fix the root cause, not the symptom.** No suppressing errors, no catch-and-ignore, no special-casing around the real problem.
4. **Prove it.** The failing test now passes, and the full suite still passes: [exact commands, verbatim — e.g. `npm test`].
5. **Check for siblings.** Does the same bug pattern exist in similar code paths? List them in the handback — do not fix them unbidden.

## Out of scope

- No refactoring beyond the minimal fix.
- [Other specific temptations to resist — nearby cleanups, drive-by improvements, dependency bumps.]

## Handback

- **Root cause, in one or two sentences.** "Fixed" without a stated root cause is not done.
- The fix: files touched and why this change addresses the cause.
- Evidence: failing-before / passing-after test output (or the manual repro results), plus full-suite results.
- Sibling locations spotted, if any.
- Suggested commit message (the user commits manually).
