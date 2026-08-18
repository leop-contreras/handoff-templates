---
name: context-audit
description: Read when asked to audit, lint, or health-check a repo's agent context (CLAUDE.md, .claude/rules, .claude/skills), when a CLAUDE.md looks bloated or instructions in it seem to be ignored, when a path-scoped rule doesn't appear to load, or before and after any docs reorganization.
---

# Context audit

Read-only doctor for a repo's agent-context files. Run the battery, report findings, **change nothing**. Fixes belong to a follow-up pass (see `context-reorg` for restructuring, or a normal edit for point fixes the user approves).

## The limits being checked (from the Claude Code docs)

1. **Size.** CLAUDE.md-type files should stay under **~200 lines**; files over that consume more context and may reduce adherence. Hard cap: **40,000 characters per memory file** — beyond it the file is flagged and Claude may not read the full content. The cap applies to *every* memory file, including each rule and skill file.
2. **A rule without `paths:` saves nothing.** "Rules without `paths` frontmatter are loaded at launch with the same priority as `.claude/CLAUDE.md`." A rule file with no `paths:` is a renamed piece of CLAUDE.md.
3. **A glob matching zero files is a silent no-op.** The rule never loads; the content is effectively lost.
4. **Skills are the only on-demand tier**, and the `description:` is what Claude matches a prompt against. A description written as a table of contents instead of a trigger condition means the skill never fires.
5. **`@path` tokens outside backticks are parsed as imports** and load at launch (or fail on nonexistent paths). Import parsing skips code spans and fenced blocks, so a backticked `` `@scope/package` `` is safe; a bare `@up.edu.mx` is not.
6. **Nested CLAUDE.md files and path-scoped rules are not re-injected after `/compact`** — anything needed in literally every session must live in the root CLAUDE.md.

## The battery

Run from the repo root (bash). Each command states its expected result — anything else is a finding.

```bash
# 1. Sizes — every memory-type file
wc -l -c CLAUDE.md                                      # < 200 lines, < 40000 chars
find .claude -name '*.md' -exec wc -l -c {} + 2>/dev/null   # each file < 40000 chars
find . -name 'CLAUDE.md' -not -path '*/node_modules/*'      # inventory nested ones; each < 40000 chars

# 2. Rule frontmatter — every rule opens with YAML and carries a non-empty paths: list
head -1 .claude/rules/*.md                              # each line must be ---
grep -L '^paths:' .claude/rules/*.md                    # must print nothing

# 3. Glob validity — for EVERY glob in every paths: list
git ls-files '<glob>'                                   # >= 1 file per glob; zero matches = broken rule

# 4. Skill frontmatter — exactly name: and description: in the top frontmatter block
#    (scan only the head — the body may quote example frontmatter in fenced blocks)
for f in .claude/skills/*/SKILL.md; do echo "$f: $(head -5 "$f" | grep -c '^name:\|^description:')"; done   # each must be 2

# 5. No @ token escaped its backticks (would be parsed as an import)
grep -rn "@[a-zA-Z]" CLAUDE.md .claude/ | grep -v '\`@'     # must print nothing

# 6. No @ imports at all (see context-reorg for why imports are banned in this layout)
grep -n '^@\|[^`]@[a-zA-Z/.~-]*\.md' CLAUDE.md              # must print nothing

# 7. Committability — new .claude files must not be silently gitignored
git check-ignore -v .claude/rules/* .claude/skills/*/SKILL.md 2>/dev/null   # must print nothing (settings.local.json excepted)

# 8. Dangling references — every "see X" must point at a section that still exists
grep -rn 'see "' CLAUDE.md .claude/                     # verify each hit by hand
```

## Judgment checks (no command can decide these)

- **Trigger-shaped descriptions.** For each `SKILL.md`, read the `description:`. Weak: `Supabase migration history and RLS post-mortems` (a table of contents). Strong: `Read when a Supabase query returns empty, a deploy 503s, a login loops, or RLS behaves unexpectedly` (the moments it should fire). Flag weak ones.
- **Grab-bag rules.** A rule covering two unrelated subsystems should split — flag any whose `paths:` globs span areas that never change together.
- **Derivable content.** The docs' exclude-list for always-loaded memory: file-by-file descriptions of the codebase, and anything Claude can figure out by reading code. Flag sections of CLAUDE.md that are pure file inventory.
- **Universality.** Anything in CLAUDE.md that only matters when touching one subsystem is a rule candidate; anything only relevant when something breaks is a skill candidate. Flag, don't move.

## Report format

A short table — one row per finding: **check | file | result (PASS / WARN / FAIL) | what to do about it** — followed by a one-paragraph verdict. Paste the actual command output for every FAIL. If the repo would benefit from a restructure (CLAUDE.md far over budget, no rules/skills tiers at all), say so and point at `context-reorg`; do not start one unbidden.
