---
name: briefing
description: Deep one-by-one walkthrough of issues, decisions, or open questions. Builds index, then each issue gets full analytical depth sequentially.
shortcuts:
  - "!поодному"
  - "!one-by-one"
---
# Skill: Briefing

A batch of 7 decisions in one response means each gets ~15% of thinking — shallow by definition. Briefing dedicates 100% of output tokens and reasoning to one issue at a time, then moves to the next.

## Algorithm

### Phase 1: Build index

The index is a dumb map — titles and pointers, not analysis. Deep thinking happens later, one issue at a time.

1. Scan source (file, conversation, agent reports) for issues needing resolution
2. Filter out already-resolved items (check by title only, don't deep-scan yet)
3. For each remaining issue: extract title + where to find context
4. Write `briefing_<case_name>.md` in the project folder
5. Link it from `plan.md` (so the next session knows there's an active briefing)
6. Show the index to user for confirmation — they may reorder, remove, or add

**Example:**

```markdown
# Briefing: instructions review
source: projects/WIP_instructions_rework/merged_output_review.md
workflow: solo

## Issues

### 1. Glossary in system prompt — needed or redundant?
- context: merged_output_review.md, item 4; core_instructions.md:39-64
- status: pending

### 2. "Be cautious about deletions" — panic tone without WHY
- context: merged_output_review.md, item 5; core_instructions.md:101
- status: pending

### 3. Observable rules table — does agent use it?
- context: merged_output_review.md, item 7; core_instructions.md:123-138
- status: pending
```

### Phase 2: Walk through

Other sessions may have already fixed some issues. Re-scanning files on every step catches this — stale analysis is worse than no analysis.

For each issue:

1. **Re-scan** source through the lens of this issue. Check current state of files/code
2. **Resolved?** — mark in index, move on
3. **Alive?** — this is the core of briefing. You are the analyst who puts a folder on the CEO's desk. The CEO was on vacation for a month — no context, no memory of prior discussions. Your job is not to present a menu of options — it's to do the hard thinking and arrive at an answer.

   **Analysis sequence** — apply explicitly, in this order:
   `[bigpic]` → `[verify]` → `[think]` → `[propose]` → `[honest]`

   **Then present:**
   - **Situation:** self-contained — reader understands the question without opening any other document
   - **Reasoning:** what you found, what it means, why one path wins. This is where the depth goes — not into formatting, but into thinking made visible
   - **Recommendation:** concrete action. The CEO reads your reasoning and says "да" — or asks one clarifying question

   If you honestly cannot resolve the issue because the answer depends on a priority only the user knows — then show the fork: what depends on what, what happens in each case. This is not an alternatives table by default — it's a last resort when the analyst has done all the thinking they can and the remaining choice belongs to the human.
4. **Wait** for decision. FULL STOP. Do not edit files, update index, or print ZERO LOOSE ENDS until the user explicitly approves or rejects. "Согласен?" from you is a question, not a decision. Only the user's reply is a decision.
5. **Act** based on workflow:
   - **solo** — fix immediately (context is hot — deferring means losing it)
   - **pair / sddg** — write task for executor (e.g. `prompt_<name>.md`), record in index
6. **Update index** — mark status, write decision
7. **Zero loose ends gate** — before moving to next issue, print this checklist in chat:

   ```
   ✅ ZERO LOOSE ENDS:
   - User approved: "<exact user quote>"
   - Files changed: <list or "none">
   - Index updated: <yes/no>
   - Decision recorded: <one-line summary>
   ```

   The "User approved" field is mandatory. You must quote the user's actual words that constitute approval. If you cannot fill this field — you skipped step 4. Stop, revert any changes, and go back to waiting.

   Do NOT present the next issue until this gate is printed and complete.

A single issue may take one message or fifty — don't move on until resolved or deferred.

**Example — clear answer:**

```
## [1/3] Glossary in system prompt — needed or redundant?

**Situation:** core_instructions.md содержит 25-строчный глоссарий (Entity Hierarchy +
таблица EN/RU). Агент уже получает context.chain с типами из workspace_info —
вопрос, даёт ли глоссарий что-то сверх этого.

**Reasoning:** Дерево Entity Hierarchy дублирует context.chain — агент получает
breadcrumb "МетаЛаб / ТехноЛаб / Duet" с типами на каждом уровне. Но таблица
EN/RU даёт то, чего chain не даёт: русские синонимы ("дело" = stream, "продукт" =
product). Без неё агент не поймёт "открой бизнес МетаЛаб". Дерево — 15 строк
дублирования, таблица — 10 строк уникального знания.

**Recommendation:** Убрать дерево, оставить таблицу EN/RU.
```

**Example — genuine fork (analyst can't resolve alone):**

```
## [2/3] Spec files — markdown or structured YAML?

**Situation:** Сейчас спеки в markdown. Появился запрос на машинное чтение спеков
(валидация, генерация). Markdown парсить ненадёжно, YAML — надёжно, но людям
читать тяжелее.

**Reasoning:** Я проверил оба варианта. Markdown спеки сейчас читают 3 агента и
человек напрямую. YAML решил бы задачу валидации, но сломал бы читаемость для
человека. Гибрид (YAML frontmatter + markdown body) — компромисс, но усложняет
структуру. Я не могу разрешить это без понимания приоритета: машинная обработка
важнее читаемости, или наоборот?

**Если читаемость важнее** → оставить markdown, валидацию делать через LLM-парсинг
**Если машинная обработка важнее** → перейти на YAML с markdown-комментариями
```

## Anti-patterns

| Don't | Why | Do instead |
|-------|-----|------------|
| Analyze during index phase | Wastes tokens on shallow pass you'll redo anyway | Index = titles + pointers only |
| Batch multiple issues in one response | Output tokens compete → shallow treatment | One issue, full depth |
| Skip re-scan before analysis | May already be fixed by another session | Check current file state first |
| Assume reader has context | Analysis gets forwarded, discussed later | Self-contained from zero |
| Present alternatives table by default | Listing options is easier than arriving at an answer — that's avoidance, not depth | Do the thinking, come with an answer. Fork only when you genuinely can't resolve without user's priorities |
