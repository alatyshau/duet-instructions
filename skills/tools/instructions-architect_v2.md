# Skill: Instructions Architect

Design AI agent instructions that work and do what's expected.

## Core Philosophy

**The only goal is a working agent.** Token cost is NEVER a criterion.

We make instructions concise NOT to save money, but because:
- Shorter = easier for agent to parse and follow
- Focused = less confusion, more stable behavior
- Clear structure = reliable execution

If longer text works better — use longer text. If verbose explanation helps — be verbose.

## When to Use

- Designing or reviewing instruction files (skills, personas, schemas)
- Restructuring the instructions workspace
- Reviewing how well agents follow instructions (signals from checkpoint)

## Edit Protocol

Instructions affect all agents on the machine. IA works in the instructions workspace (path from `instructionsPath` in `workspace_info`).

| Action | Who |
|--------|-----|
| New file — create in place | IA |
| Existing file — create `*_v2` next to original | IA |
| Merge (replace original with v2) | Human |
| Commit | Human |

IA never deletes, overwrites, or commits instruction files.

## How to Write Instructions

### Explain the Why

LLMs are smart. They have good theory of mind and respond better to understanding motivation than to rote rules. **Every instruction should carry its reason.**

If you find yourself writing ALWAYS or NEVER in all caps, or stacking rigid MUSTs — that's a yellow flag. Reframe: explain the reasoning so the agent understands why this matters. That produces more robust behavior than any amount of shouting.

Bad: `NEVER use markdown links for paths`
Good: `Use backticks for paths (`` `skills/modes/planning.md` ``), not md-links — md-links break when instructions are loaded from varying nesting depths`

**Leading WHY pattern:** The strongest instructions open with the reason, and the rule follows naturally. The reader understands the motivation before encountering the constraint — which makes the constraint feel obvious rather than imposed.

Bad: `Keep plan.md to one screen. The user reads it to understand the full picture.`
Good: `The user reads plan.md to understand the full picture without opening other files. Keep it to one screen.`

This pattern comes from `core_instructions.md` L7+ rules — study how each rule's first sentence is a WHY that makes the action self-evident.

### Progressive Disclosure

Skills use a three-level loading system. Write with this in mind:

1. **YAML frontmatter** (name, description, trigger/noTrigger) — always in context via workspace_info catalog. The description and trigger fields determine whether the agent loads the skill. Make them specific about WHEN to use, not just WHAT it does.
2. **Skill file body** (<500 lines ideal) — loaded when skill activates. Everything the agent needs to start working.
3. **Companion resources** (unlimited) — loaded on demand. Reference files, prompts, templates. Point to them from skill file with clear guidance on WHEN to read.

If approaching 500 lines, add hierarchy: split into a main file with pointers to reference files.

### Keep It Lean

After writing, reread with fresh eyes and cut anything that isn't pulling its weight. Signs of bloat:
- Context the agent already has from its system prompt (L7+ rules, project management, spec-driven development, glossary — all always present). Don't restate rules like `[tradeoff]`, `[verify]`, `[honest]`, project folder conventions, or plan.md structure
- Knowledge the agent has built-in (don't teach it Python — teach it YOUR conventions)
- Descriptions of behavior instead of prescriptions ("X is important" → "When X, do Y")
- Rules that exist "just in case" with no concrete scenario

### Generalize, Don't Overfit

Instructions get used across many sessions with different contexts. If you're fixing a specific failure, ask: is this a pattern or a one-off? The fix should address the pattern, not the single case. Overly specific patches accumulate into brittle instruction sets.

### Examples > Explanations

An example of good output teaches more than a paragraph describing what good output looks like. When a rule is hard to articulate precisely, show 2-3 examples instead.

## Writing Skills

Schema: `schemas/skill_file.md`. Read it before writing.

**What makes a good skill file:**

- **Starts with the job.** First line after title = what the agent should be able to DO after reading this.
- **One read = ready to work.** If the agent needs to read 3 more files before it can start — the skill is too fragmented.
- **Specific to your context.** YOUR conventions, YOUR project's patterns, YOUR preferred approach.

**Complex skills** use a companion folder for resources and a scripts directory for code:

```
skills/tools/scriptor.md          ← skill file (loaded by agent)
skills/tools/scriptor/            ← resources: prompts, templates
scripts/scriptor/                 ← Python code
```

Markdown and Python don't mix — skills in `skills/`, scripts in `scripts/`.

## Writing Personas

A persona defines WHO the agent is for the session.

Good personas have:
- **Clear focus area** — what domains this persona is best at
- **Behavioral traits** — how it thinks, responds, pushes back
- **Boundaries** — what this persona does NOT do (defers to other personas)

Bad personas are just skill lists with a name on top. If removing the name changes nothing — it's not a persona.

## Maintaining the Catalog

The catalog is built dynamically from YAML frontmatter in each file. Every persona and skill must have valid frontmatter (name, description). `index.json` declares which folders to scan.

When IA adds a new file, it ensures the file has proper YAML frontmatter. If adding to a new category folder, it also updates `index.json`. This is one atomic action.

## Checklist

Before editing an existing file:
- [ ] Read the current version first
- [ ] Is there a schema for this file type? (`schemas/`)
- [ ] What concrete problem am I fixing? (no edits "for cleanliness")

Before writing content:
- [ ] Does the agent already know this? (don't repeat built-in knowledge)
- [ ] Example or explanation? (prefer example)
- [ ] Did I explain the why behind every non-obvious rule?
- [ ] Can I cut anything that isn't pulling its weight?

When adding a new skill:
- [ ] Skill file in `skills/<category>/` following `schemas/skill_file.md`
- [ ] YAML frontmatter with name, description (and shortcuts/trigger if applicable)
- [ ] If complex — companion folder for resources, scripts in `scripts/`

## Anti-patterns

| Don't | Why not | Do instead |
|-------|---------|------------|
| Stack ALWAYS/NEVER/MUST rules | Rigid rules break on edge cases; agents respond better to reasoning | Explain the why, let agent generalize |
| Repeat bootstrapper or index.md content | Bloat; agent already has it in context | Focus on what only THIS file teaches |
| Teach built-in knowledge | Waste of tokens and attention | Focus on YOUR specific conventions |
| Write narrow patches for one-off failures | Accumulates into brittle instructions | Fix the pattern, not the instance |
| Describe behavior ("X is important") | Agent needs to know what to DO | Prescribe: "When X, do Y" |
| Deeply nested references (>2 levels) | Agent loses track, costs tokens | Flat: one hop from main file |
| Put Python next to markdown | Confuses skill content with code | `skills/` for md, `scripts/` for py |
| Use md-links for instruction paths | Break at varying nesting depths | Backticks: `` `skills/tools/checkpoint.md` `` |
| Forget YAML frontmatter on new files | Catalog won't pick them up | Always add name + description frontmatter |
