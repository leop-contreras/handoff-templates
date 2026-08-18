---
name: context-reorg
description: Read when a repo's CLAUDE.md is too big (over ~200 lines or 40,000 chars), when asked to split agent docs into path-scoped rules and skills, when instructions are being ignored because context is bloated, or when a context-audit recommended a restructure.
---

# Context reorg

Split a monolithic CLAUDE.md into the three-tier layout the Claude Code docs prescribe. This skill carries the reusable methodology; the repo-specific facts (section maps, line numbers, globs) are derived fresh each time — never reused from a previous run.

## The three tiers

| Tier | Lives at | Loads | Holds |
|---|---|---|---|
| CLAUDE.md | repo root | every session | the truly universal stuff, kept **under ~200 lines**: what the app is, tech stack, how to run it, conventions, non-negotiable invariants |
| Rules | `.claude/rules/*.md` with `paths:` frontmatter | when Claude touches a matching file | anything specific to a subsystem, directory, or language — **this is the tier that actually cuts per-session context** |
| Skills | `.claude/skills/<name>/SKILL.md` | on demand, matched against the prompt via `description:` | diagnostic knowledge (gotchas, post-mortems), project history, workflows that are only sometimes relevant |

Rule frontmatter shape:

```
---
paths:
  - "src/api/**"
  - "src/services/**"
---
```

Skill frontmatter is exactly `name:` + `description:`. Write every `description:` as a **trigger condition** (the situations in which it should load), never a table of contents. Do not set `disable-model-invocation: true` on knowledge skills — the point is that Claude surfaces them itself.

## Five facts that constrain every placement decision

1. **A rule without `paths:` saves nothing** — it loads at launch with the same priority as CLAUDE.md. Every rule file created must carry a `paths:` list, and every glob must match at least one real file (verify with `git ls-files '<glob>'`).
2. **Skills are the only tier that stays out of context until needed.**
3. **`@path` imports are banned in this layout.** Imports load at launch alongside CLAUDE.md — "helps organization but doesn't reduce context." Reducing context is the entire purpose, so anything worth importing is by definition always-loaded content that belongs inline, counted against the 200-line budget. Corollary: keep every literal `@` (`@scope/pkg`, `@domain.com`) inside backticks, or it parses as an import.
4. **Block-level HTML comments in CLAUDE.md cost zero context** (stripped before injection) — usable for maintainer notes and pointer stubs that would otherwise eat the line budget.
5. **Path-scoped rules and nested CLAUDE.md files are not re-injected after `/compact`** — an accepted trade-off, and the reason anything needed in *every* session stays in CLAUDE.md proper.

## Non-negotiable working rules

- **No content may be lost.** Every line of the original ends up in the trimmed CLAUDE.md, a rule, or a skill. Historical/RESOLVED narrative is relocated, never pruned. Prove it with line accounting: redistributed total ≥ original total (frontmatter adds lines).
- **Prose moves verbatim.** No rewording, summarizing, or "improving". Permitted edits only: adding frontmatter, promoting headings so the destination hierarchy is coherent, rewriting a cross-reference whose target moved, and correcting a factually wrong file path (flag it when you do).
- **Cross-references are rewritten, not left dangling.** Grep for `see "`, `below`, `above`, and cross-repo pointers; every hit whose target moved must name the new file.
- **Leave a one-line pointer** in CLAUDE.md naming where each relocated section went.
- **No `docs/` folder.** Prose that fits neither a path-scoped rule nor a workflow is still a skill with a trigger-shaped description — a `docs/*.md` that nothing loads is deletion with extra steps. (Canonical case: "project history / where this is going" becomes a skill triggered by planning questions and "why don't we just do X".)
- **File-by-file tables split row-by-row**: rows that are really a warning or rationale go verbatim into the subsystem rule covering that file; purely descriptive rows collapse into a short entry-point list (~15 lines) in CLAUDE.md. Nothing is dropped.
- **Auto-generated blocks stay put** (e.g. Next.js's `<!-- BEGIN:nextjs-agent-rules -->` block, re-added by `next dev`). Moving them only re-creates an uncommitted change.

## Procedure

**Phase 0 — decide the vehicle.** If the monolith is over ~500 lines, spans multiple repos, or the user wants the work done by a cheaper executor: do NOT execute directly. Produce a handoff from `handoff-full.md` in the handoff-templates repo (`C:\handoff-templates`; a completed example of exactly this job is `c:\pwnteras\handoff-docs-reorg.md`), embedding this skill's rules plus the repo-specific manifest. Otherwise execute in-session with the same phases.

**Phase 1 — audit and manifest (nothing moves).** Read the monolith *in ranges* (a 100KB+ file read whole wastes the window — build a section map with `grep -n "^## "` first). Produce a manifest mapping **every line range to exactly one destination** — no gaps, no overlaps, show the arithmetic. Verify every proposed glob against the real tree. Draft every skill description. **Present the manifest for approval before writing anything** — a bad line in the plan becomes hundreds of bad lines moved wrong.

**Phase 2 — execute the split.** Create `.claude/rules/` and `.claude/skills/`, move prose verbatim per the manifest, add frontmatter, trim CLAUDE.md, leave pointers. Line numbers shift the moment you edit — slice top-to-bottom in one pass or re-derive offsets before each cut.

**Phase 3 — cross-reference repair.** Sweep for the reference patterns above; fix both directions if repos reference each other.

**Phase 4 — verify.** Run the full `context-audit` battery. Additionally: line-accounting proof, `git status --short` showing only `CLAUDE.md` and `.claude/` paths (any source file in the diff is a failure — this pass changes zero code), and new `.claude/` files appearing as untracked (not silently gitignored).

Checkpoint after each phase with evidence pasted, and suggest a commit message; the user commits.

## The bar

A developer new to the repo reads the trimmed CLAUDE.md in two minutes, knows what the app is and how to run it, and trusts that the detail they need will load when they open the relevant file.
