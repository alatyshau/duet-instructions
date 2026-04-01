---
name: project-revision
description: Deep audit and restructuring of a project folder. Use when a project folder has grown messy, plan.md is unreadable or outdated, files have accumulated without clear structure, or the user explicitly asks to clean up / review a project folder. Also trigger when starting work in a project folder that feels disorganized — stale TODOs, unclear status, files that don't connect to plan.md.
noTrigger: Do not trigger for routine plan.md updates during normal work. The bootstrapper's Project Management section covers day-to-day maintenance. This skill is for when things have drifted and need reconstruction.
shortcuts: ["ревизия", "!ревизия"]
---

# Skill: Project Revision

A project folder exists so the user can open plan.md and understand the full picture in 30 seconds. When that stops working — when plan.md is a wall of checkboxes, files have piled up without links, completed work isn't reflected — the folder has drifted from its purpose. This skill reconstructs clarity.

## The core problem

Drift is natural. Every session adds artifacts — design docs, reviews, notes. Over weeks, plan.md accumulates implementation details that mattered in the moment but obscure the big picture now. The user opens the folder and feels "what is all this?" — that's the signal.

The fix is never reformatting. Reformatting preserves the mess in a prettier layout. The fix is **reconstruction**: read everything, understand what actually happened, and write a new plan.md from that understanding.

## How to do a revision

This is deep work, not a quick pass. A real revision of a complex project folder can take 30+ minutes of autonomous work. Don't cut corners for speed — a shallow revision creates new drift instead of fixing the old one.

### 1. Preserve the old state

Rename `plan.md` → `plan_old.md`. The old file is your reference and the user's safety net — they can compare old and new to verify nothing important was lost.

### 2. Read everything

Every file in the project folder. Every design doc. If this is a product with a repo — specs, READMEs, recent git log. You need to understand what actually happened, not what the old plan.md claims happened. Old plan.md may be wrong, outdated, or incomplete — it's a symptom, not a source of truth.

### 3. Diagnose

Look for specific symptoms:
- Is plan.md bloated? (over one screen, implementation details, stale checkboxes)
- Are there orphan files not linked from anywhere?
- Are there completed phases not reflected in ЧТО СДЕЛАНО?
- Is the goal still accurate or has the project evolved past it?
- Are design docs still relevant or superseded by implementation?
- Are files piling up at root without logical grouping? (5+ files at root is a signal — a flat pile is opaque to anyone returning months later, especially in archive)

### 4. Reconstruct the structure

The project may not have clear phases. Your job is to find them — from chronology (git log, file dates), content clusters, and logical dependencies. Propose a grouping to the user: "I see three clusters of work, here's how I'd structure them — agree?"

Report in chat, not in files. Wait for approval before touching anything.

```markdown
## Ревизия: <folder name>

**Состояние:** <one sentence — how bad is it?>

**Структура** (предлагаемые фазы):
- Фаза 1 — <goal>: <files involved>
- Фаза 2 — <goal>: <files involved>
- ...

**Проблемы:**
- ...

**Организация файлов:**
- <proposed subfolder structure, e.g. deliverable/, analysis/, input/>
- <rationale: what goes where and why>

**Предложение:**
- plan.md: rewrite from scratch
- Файлы: <archive / rename / restructure / reorganize into subfolders>

Одобряешь?
```

### 5. Work through phase by phase

After the user approves the structure — go deep into each phase one at a time:
- Read the phase's files thoroughly
- Understand what actually happened and why
- Clean up files (rename stale docs with `YYMMDD_` prefix, link active ones)
- Write the milestone for this phase in the new plan.md
- Show the user — get approval before moving to the next phase

**Reorganize files into subfolders.** Almost always worth doing — project folders go to archive, and clarity matters more there than during active work. Group files by role: outputs (deliverables, analysis results), inputs (reference materials), research. `plan.md` always stays at root — it's the entry point. After moving files, update all links in plan.md.

Don't rush to the next phase. Each one is its own knot to untangle.

### 6. Assemble the final plan.md

After all phases are processed — the new plan.md should already be complete. Follow the bootstrapper's format:

- Goal at the top — vivid, human, no unexplained terms
- `## ЧТО СДЕЛАНО` — narrative milestones: `**Фаза N — <goal>:** <what happened>. → links to details`
- `## ЧТО ДАЛЬШЕ` — remaining work

One screen. If it doesn't fit — something belongs in a separate file.

After user approves the final result — archive `plan_old.md` with `YYMMDD_` prefix or delete it.

## Verify your work

Before presenting the result, check:
- plan.md fits on one screen
- ЧТО СДЕЛАНО tells a narrative, not a checklist of implementation details
- ЧТО ДАЛЬШЕ is actionable — concrete next steps, not vague intentions
- Every file in the folder is linked from plan.md or archived
- The user can understand the full picture without opening other files

## What goes wrong

Shallow reading leads to shallow rewrites. If you skim the files and rephrase the old plan.md — you haven't done a revision, you've done formatting. The whole point is that you bring fresh understanding from primary sources.

Archiving files you don't understand destroys context. The user may have kept a failed design doc because it documents why an approach was rejected — that's valuable. When in doubt, ask.

Copying ✅/❏ checkboxes into the new plan.md reproduces the problem you're solving. Checkboxes are implementation details for agents. The user needs narrative milestones.
