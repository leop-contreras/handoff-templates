# Handoff templates

Two things live in this repo:

1. **Three handoff templates** for a two-model pipeline: **you** write a detailed prompt → a **high-tier model** (Opus/Fable, with repo access) restructures it into one of these templates → a **cheaper executor** (Sonnet, or Opus at low effort) implements from the finished handoff in a fresh session.
2. **Three context skills** (`skills/`) that make the CLAUDE.md organization proven in the pwnteras repos — thin root file + path-scoped rules + on-demand skills — repeatable on any project. See [Context skills](#context-skills) below.

The templates are built around one idea: the restructuring step is a *compression* step. Information gets lost there by default, never gained — unless you force the formatter to preserve your specifics and enrich the doc with things the executor is bad at finding on its own (file paths, verification commands). Every rule in these templates exists to serve a weaker executor: concrete paths, runnable checks, checklists instead of prose, and zero filler.

## Picking a template

| Situation | Use |
|---|---|
| You could describe the diff in one sentence | **No template.** Just prompt the executor directly — a handoff doc for a typo fix is pure overhead. |
| Single-phase change, no schema/data impact, one sitting of work | [handoff-light.md](handoff-light.md) |
| Multi-phase feature; touches data, schema, or several layers; needs checkpoints | [handoff-full.md](handoff-full.md) |
| Something is broken and needs fixing | [handoff-bugfix.md](handoff-bugfix.md) |

Don't upgrade "to be safe." A frontier model filling a template will *fill it* — forcing a small change into the Full template produces invented content for sections that don't apply, and that fabricated detail actively misleads the executor. Each template's formatter rules tell the model to delete sections rather than pad them, and the Full and Light templates additionally instruct it to say so if the task belongs in the other one.

## Prompting the formatter

Give Opus/Fable three things: the template file, your raw prompt, and repo access. A prompt that works:

```
Read handoff-templates/handoff-full.md and follow its FORMATTER RULES
comment exactly. Restructure my prompt below into that template.

You have repo access: verify every file path you write, and add the
relevant paths I didn't mention. Preserve my concrete details verbatim
— names, values, copy text, constraints. Where I'm ambiguous, insert
[NEEDS CLARIFICATION: ...] instead of choosing for me. Delete sections
that don't apply. Output only the finished handoff, ready to hand to
another model.

<my-prompt>
...your detailed prompt here...
</my-prompt>

Ask questions, then write the handoff as file, not in chat.
If any [NEEDS CLARIFICATION] are left in the file after making the file, list them with bullet points in chat.
```

What makes the difference:

- **Repo access matters more than eloquence.** The single highest-value thing the formatter adds is a verified map of where the change lives. Without repo access it can only reformat your words — mark that case by telling it to tag guessed paths `[VERIFY PATH]` (the templates already instruct this).
- **Your raw prompt should still be detailed.** The template structures information; it doesn't create it. Vague in, vague out — just better formatted.
- **Never let it resolve ambiguity for you.** `[NEEDS CLARIFICATION]` markers are the mechanism. You answer them; the formatter doesn't guess, and the executor is instructed to stop if any survive.

## Review the handoff before you hand it off

A bad line in a plan becomes hundreds of bad lines of code — reviewing the handoff is the highest-leverage minute in the whole pipeline. Thirty-second checklist:

1. Are the file paths real? Spot-check one or two. Any `[VERIFY PATH]` left? Verify them.
2. Are all `[NEEDS CLARIFICATION]` markers resolved? Answer them and have the formatter (or you) edit them in.
3. Are the verification commands real commands from this repo, with expected results?
4. Any section that smells like filler — generic content in Data model or Risks that doesn't match the task? Delete it.
5. Did your specifics survive? Check that exact values, names, and copy text you wrote appear verbatim, not paraphrased.
6. Is the formatter-rules comment block gone from the top?

## Running the executor

Start a **fresh session** — clean context focused only on the handoff — and point it at the doc: *"Read handoff-X.md in full and execute it."*

What to expect, by template:

- **Full:** the executor stops at the end of every numbered phase with a short report — what was done, files touched, exit-criteria evidence (actual command output, not claims), and a suggested commit message. **You make the commit yourself.** The executor is instructed not to start the next phase until the previous phase's commit exists, so an uncommitted phase halts the pipeline by design.
- **Light:** one pass, ending with a handback — files touched, verification output, suggested commit message. You commit.
- **Bugfix:** the executor must state the **root cause** and show failing-before / passing-after evidence. "Fixed" without a root cause isn't done. It also lists sibling locations where the same bug pattern may exist — it won't fix those unbidden; triage them yourself.

In every template the executor is told to **stop and flag** when reality contradicts the doc (wrong version, missing file, misbehaving API) instead of improvising. When it flags, fix the handoff doc — don't just answer in chat — so the doc stays the single source of truth for the rest of the run.

## Context skills

`skills/` holds three Claude Code skills that turn the CLAUDE.md reorganization done on `pwnteras-landing` and `learning-path-demo` (via `handoff-docs-reorg.md`) into a reusable capability. The idea: that handoff was ~80% reusable methodology and ~20% repo-specific detail (line maps, globs). The skills keep the 80%; the agent re-derives the 20% fresh against whatever repo it's pointed at.

All three enforce the same three-tier layout from the Claude Code docs:

- **`CLAUDE.md`** (root) — the truly universal stuff, under ~200 lines. Loaded every session.
- **`.claude/rules/*.md`** with `paths:` frontmatter — subsystem-specific knowledge that loads only when Claude touches matching files. The only tier that actually cuts per-session context.
- **`.claude/skills/*/SKILL.md`** — gotchas, post-mortems, history, workflows; loaded on demand when the prompt matches the skill's trigger description.

### The three skills

| Skill | What it does | When you'd say |
|---|---|---|
| **context-audit** | Read-only health check: file sizes vs the 200-line/40KB limits, rules missing `paths:`, globs matching zero files, weak skill descriptions, unbackticked `@` tokens, dangling "see X" references. Reports; changes nothing. | "audit the context files in this repo" |
| **context-reorg** | The full split methodology for an oversized CLAUDE.md: audit → approved manifest → verbatim moves → cross-reference repair → verification, with the no-content-lost rule. For big jobs (500+ lines, multi-repo) it doesn't execute directly — it writes a handoff from `handoff-full.md`, feeding this repo's normal pipeline. | "this CLAUDE.md is way too big, split it" |
| **context-init** | Greenfield scaffold: thin CLAUDE.md skeleton + empty rules/skills dirs, plus a **filing policy** section baked into the new CLAUDE.md that tells every future agent session which tier new knowledge goes in. This is the prevention half — the reorg you never have to run. | "set up agent docs for this new project" |

They chain: **init** prevents, **audit** detects, **reorg** cures — and reorg hands big jobs back to the handoff templates.

### Install and use

They're plain personal skills, not a plugin — three files don't need a marketplace layer, and personal skills apply across all projects automatically. They're wired into `~/.claude/skills/` as directory junctions pointing back here, so editing a file in `skills/` updates the live skill with no re-install:

```powershell
# already done on this machine; for a new machine:
foreach ($s in 'context-audit','context-reorg','context-init') {
  New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\$s" -Target "C:\handoff-templates\skills\$s"
}
```

Then in any repo, either invoke one directly (`/context-audit`) or just describe the situation ("this CLAUDE.md is huge") — the trigger descriptions are written so Claude loads the right one on its own. If these ever need sharing beyond one machine, the folder converts to a plugin marketplace with one `marketplace.json` (see [dashed/claude-marketplace](https://github.com/dashed/claude-marketplace) for the pattern).

Borrowed from prior art: the audit-battery-with-expected-outputs style comes from Anthropic's ["give Claude a way to verify its work"](https://code.claude.com/docs/en/best-practices); manifest-before-moving and human review of the plan from [HumanLayer's ACE-FCA](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md); the scaffold-path-scoped-rules-from-day-one idea from the [komluk-scaffolding](https://www.claudepluginhub.com/plugins/komluk-scaffolding) plugin's `/init-rules`; the tier decision table from Anthropic's [steering Claude Code](https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more) post.

## Why the templates look like this

- Verification sections with runnable pass/fail checks, and evidence over assertions: [Anthropic — Claude Code best practices](https://code.claude.com/docs/en/best-practices) ("give Claude a way to verify its work" is their top-ranked practice).
- Pinned `file:line` context and phase-wise plans reviewed by a human: [HumanLayer — Advanced Context Engineering for Coding Agents](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md).
- `[NEEDS CLARIFICATION]` markers, P1–P3 priorities, and acceptance criteria per behavior: [GitHub spec-kit templates](https://github.com/github/spec-kit/blob/main/templates/spec-template.md).
- Short, concrete instructions and tight checklists instead of philosophy, because prompts tuned for frontier models regress on smaller ones: [Arize — matching frontier models with a smaller model](https://arize.com/blog/how-to-ditch-your-frontier-model-for-an-slm/).
