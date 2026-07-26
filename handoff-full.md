# Handoff: [short, descriptive title of the feature/change]

<!-- ==================== FORMATTER RULES ====================
You are restructuring a human-written prompt into this template.
DELETE this entire comment block from the finished handoff.

1. PRESERVE VERBATIM every concrete detail from the source prompt:
   names, numbers, copy text, library choices, constraints. Do not
   round specifics into general language.
2. ENRICH, don't just reformat. Open the repo and pin real file
   paths (file:line where it helps) in "Where this lives" and in
   the phases. Mark any path you could not verify as [VERIFY PATH].
3. Where information is missing or ambiguous, write
   [NEEDS CLARIFICATION: <specific question>]. Never invent a
   value, convention, or requirement to fill space.
4. If a section does not apply to this change, DELETE the whole
   section. Never write filler.
5. Keep checklists as checklists — the executor works by checkbox.
6. If this change is single-phase with no data/schema impact, it
   belongs in the Light template instead. Say so rather than
   padding this one.
========================================================== -->

## Read first

- Read in full before doing anything else: [name the doc(s) that hold current architecture/context — e.g. `CLAUDE.md`, a README, ADRs]
- Already settled there — do not re-derive or re-investigate: [data model, API shape, key conventions, commands, prior migration history, ...]
- This handoff is a change **on top of** the existing architecture, not a rewrite. Unless this doc says otherwise, these stay exactly as-is: [e.g. the auth flow, the deployment pipeline, the existing component library].

## Working rules

- **Checkpoints:** stop at the end of every numbered phase. Post a checkpoint report (format at the bottom of the phases section) and wait for a go-ahead.
- **Commits:** the user makes all commits manually. At each checkpoint, suggest a commit message for the phase. **Do not start the next phase until the previous phase's work is committed** — check `git status` / `git log` first; if the phase's changes are still uncommitted, stop and wait.
- **Wrong assumptions:** if anything documented here turns out to be wrong against the real code (a library version differs, a file doesn't exist, an API misbehaves), stop and flag it. Do not guess and continue.
- **Unresolved markers:** if any `[NEEDS CLARIFICATION]` below is still unresolved when you reach it, stop and ask — never pick a value yourself.

[Optional: if specific parts of this work can safely run in parallel or should be delegated to subagents, say which and why. Otherwise DELETE this line — the default is one agent, phases in order.]

## What we're building

[2–5 plain-language sentences: what the old behavior was, what the new behavior will be, what changes for the user or system. Someone reading only this section should know what's being built.

Name any library, framework, or tool that must be used for a significant part of the work, and why (e.g. "use X instead of hand-rolling this — it already solves the hard part"). If a choice must be confirmed against what's already installed or in use in the repo, say so explicitly.]

## Where this lives (current state)

[Map the change onto the actual codebase so the executor doesn't rediscover it. Real paths, verified against the repo.]

| Area | Path | What's there now / what changes |
|---|---|---|
| [e.g. settings UI] | `src/components/Settings.tsx` | [current behavior; what this handoff does to it] |
| [e.g. API route] | `src/app/api/foo/route.ts` | [...] |

[Also list anything the executor should NOT need to read or touch — e.g. "everything under `src/legacy/` is untouched; don't read it." This protects the executor's context budget.]

## Functional spec

[One subsection per logical piece of the feature — as many or as few as it really has. Priority key: **P1** = core, the handoff fails without it. **P2** = should ship in this pass. **P3** = only if nothing above is at risk; skipping is acceptable but must be flagged. If a phase runs into trouble, protect P1 first and say so at the checkpoint.]

### [Piece name] — [P1 | P2 | P3]

- [ ] [Behavior described precisely enough that two different implementers would build the same thing]
- [ ] [Sequences spelled out step by step in order — "on click, then on hold, then on release" — not abstractly]
- [ ] [Known edge cases and invariants stated explicitly: empty list, two items at the same position, network failure mid-action]
- [ ] [Undecided values marked: [NEEDS CLARIFICATION: exact debounce ms?]]
- [ ] [Intentionally minimal parts marked directly: "no animation", "no loading state", "internal tool — no polish"]

### [Next piece] — [P1 | P2 | P3]

- [ ] ...

## Out of scope for this pass

[Specific adjacent things someone might reasonably assume are included but that must NOT be built unless explicitly asked. Be concrete — "no auto-layout", not "don't overdo it".]

- ...
- ...

## Data model / system changes

[DELETE this section if the change touches no schema, API contract, state shape, or file format. Otherwise: the exact changes — which tables/fields/types are added, removed, or changed, and how. Name the existing conventions (naming style, mapping layers, file locations) the new structures follow; a second, parallel convention must not be introduced.

Call out any data that must NOT be regenerated, renamed, or disturbed (IDs, historical records, user data, anything with downstream dependents) and what depends on it staying stable. State the safe migration approach (additive/ALTER-style vs. destructive drop-and-recreate) if the wrong way would look reasonable.

If part of the existing data flow does NOT change, say so — it saves the executor from second-guessing something that's already fine.]

## Known risks & landmines

[DELETE if genuinely none. Things that could bite: fragile modules, race conditions, hidden couplings, deceptively-named things, and past attempts that failed and why.]

- ...

## Verification

[The executor must be able to prove the work is correct without the user watching. Every check must be runnable and have an expected result.]

- **Commands** (verbatim, even if also documented elsewhere): [e.g. `npm test`, `npm run lint`, `npx tsc --noEmit`, `npm run build`]
- **Acceptance checks** — all must pass before handback:
  - [ ] [One per P1/P2 behavior, as command + expected output or Given/When/Then — e.g. "run `npm test` — all pass, including the new tests for X"]
  - [ ] [e.g. "load `/settings`, toggle Y — the value persists after reload"]
  - [ ] [e.g. "screenshot of Z matches the referenced design"]
- **Evidence rule:** show the actual command output / screenshot at each checkpoint and in the handback. Never assert "works" without evidence.

## Code quality bar

- Match the existing codebase's style, naming, and file organization. No parallel conventions.
- Build only what this doc specifies — no speculative abstraction, no "future-proofing".
- Comment intent and constraints, not mechanics. If code needs heavy comments to be safe to edit, simplify the code instead.
- New logic goes in [new, clearly-named files under `<dir>` / the existing files listed in "Where this lives" — pick one]. Hub/entry files stay thin.
- Keep functions and files small and composable, but not fragmented to the point of being hard to follow.

**The test:** a competent developer new to this repo should be able to read [the architecture doc] and skim the new files once, then make a small change without asking the author anything.

## Documentation updates

[DELETE if truly none. Every doc that gets updated as part of this work — not deferred to "later". For each: the doc → the specific sections to update → audience/tone if it matters. If a doc may not exist yet, check first; a minimal fallback is fine — don't create something elaborate.]

- [ ] `[doc path]` — [sections: e.g. architecture overview, key files table, anything describing the old behavior this replaces]

## Implementation phases

[Ordered by real dependencies (data layer → API → UI). Each phase: a short title, 1–2 sentences of scope, what's explicitly deferred to a later phase, and **exit criteria** — the specific checks from the Verification section that must pass before the checkpoint.]

1. **[Phase title]** — [scope. Deferred to phase N: ...]
   - Exit: [command(s) + expected result]
2. **[Phase title]** — [...]
   - Exit: [...]

**Checkpoint report format (post at the end of every phase, then stop):** 10 lines max — what was done, files touched, exit-criteria evidence (paste the output), suggested commit message, any deviations or flags. Wait for the go-ahead and for the commit to exist before starting the next phase.

## Handback (final report)

When every phase is committed, post a final report:

- What changed, one line per phase
- Full list of files created/modified
- Every verification command run, with its output
- Anything skipped, deferred, or deviating from this doc — stated explicitly, even if minor
- Optional: up to 3 suggested follow-ups. Do not implement them.
