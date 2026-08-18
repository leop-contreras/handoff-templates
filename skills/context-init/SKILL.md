---
name: context-init
description: Read when starting agent docs for a new or undocumented project, when a repo has no CLAUDE.md, when asked to scaffold .claude rules/skills structure, or when setting up a project so its context files won't grow into a monolith.
---

# Context init

Scaffold a new repo's agent context in the three-tier layout from day one, so a `context-reorg` is never needed. Curing a 2,000-line CLAUDE.md costs a multi-phase migration; preventing it costs a filing-policy paragraph.

## What to create

```
CLAUDE.md               # thin, from the skeleton below — only what is actually known today
.claude/rules/          # created empty, or with the first obvious path-scoped rule
.claude/skills/         # created empty
```

Do not pre-create empty rule or skill files "for later" — a glob matching nothing and a skill with no content are silent no-ops. The tiers get populated when there is something to put in them. If one subsystem already has non-obvious conventions worth writing down (an API layer, an auth flow), write that first rule now with real `paths:` globs verified via `git ls-files '<glob>'`.

## CLAUDE.md skeleton

Fill only the sections that have real content today; delete the rest — a scaffold with placeholder prose is worse than a short file. Never let the file pass ~200 lines or 40,000 characters.

```markdown
# CLAUDE.md

Guidance for working in this repo. Read this first — it captures the
architecture and the non-obvious decisions so you don't have to re-derive
them each session.

## What this is

[2–5 sentences: what the app is, its main surfaces, who uses it.
State non-negotiable invariants here — the rules that are never safe
to ignore (naming prefixes, security boundaries, data that must not
be touched).]

## Tech stack

[Frameworks + versions, in one list. Only what an agent must know to
write idiomatic code here.]

## Running it

[The exact commands: dev, build, test, deploy. Verbatim, runnable.]

## Conventions

[House style that differs from tool defaults. If it matches defaults,
leave it out — Claude can derive it.]

## Where knowledge goes (filing policy)

New knowledge learned while working here gets filed by tier — never
appended to this file by default:

- **Applies to every session** (commands, invariants, stack): add it
  here, keeping this file under 200 lines. If a section outgrows the
  budget, move the detail to a rule and leave a one-line pointer.
- **Specific to one subsystem, directory, or language**: add it to a
  rule in `.claude/rules/<subsystem>.md` with `paths:` frontmatter
  scoped to the files it's about. A rule without `paths:` loads every
  session and belongs here instead.
- **Only sometimes relevant** (gotchas, post-mortems, project history,
  workflows): add it to a skill at `.claude/skills/<name>/SKILL.md`
  whose `description:` names the situations it should load in — a
  trigger condition, not a table of contents.
- Keep literal `@` tokens (`@scope/pkg`, `@domain.com`) inside
  backticks — a bare one is parsed as an import. Use zero `@path`
  imports: they load at launch and save no context.
```

The filing-policy section is the load-bearing part of this skill — it is what keeps the structure maintained by every future agent session, not just created once. Include it verbatim (adjust nothing but formatting).

## Frontmatter reference

Rule (`.claude/rules/<name>.md`):

```
---
paths:
  - "src/api/**"
  - "src/services/**"
---
```

Skill (`.claude/skills/<name>/SKILL.md`):

```
---
name: kebab-case-name
description: Read when [the situations this should load in].
---
```

Naming: kebab-case, subsystem-named, one convention across the repo.

## Finishing

- Run the `context-audit` battery on the result — a fresh scaffold should pass everything.
- Confirm the new files are committable: `git status --short` shows them untracked, `git check-ignore` is silent.
- Suggest a commit message; the user commits.
- As the project grows, `context-audit` periodically is the maintenance loop; `context-reorg` is the cure if the policy was ignored long enough to matter.
