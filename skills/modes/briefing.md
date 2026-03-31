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

### Phase 1: Build index + choose fixation strategy

The index is a dumb map — titles and pointers, not analysis. Deep thinking happens later, one issue at a time.

1. Scan source (file, conversation, agent reports) for issues needing resolution
2. Filter out already-resolved items (check by title only, don't deep-scan yet)
3. For each remaining issue: extract title + where to find context
4. **Choose fixation strategy** (see below). Assess whether issues are independent or interconnected. Write the strategy in the briefing file header.
5. Write `briefing_<case_name>.md` in the project folder
6. Link it from `plan.md` (so the next session knows there's an active briefing)
7. Show the index + chosen strategy. Agent owns the list and order. User approves the strategy — it's the only decision at this phase.

**Example output at this step:**

```
5 вопросов:

| # | Вопрос | Контекст |
|---|--------|----------|
| 1 | Glossary в system prompt — нужен или дублирует? | core_instructions.md:39-64 |
| 2 | "Be cautious about deletions" — паника без WHY | core_instructions.md:101 |
| 3 | Observable rules table — агент её использует? | core_instructions.md:123-138 |
| 4 | Spec gate — когда читать спеку? | review.md item 3 |
| 5 | Plan.md растёт — нужен ли лимит? | review.md item 6 |

Стратегия фиксации: **artifact → design_review.md** (вопросы связаны —
решения формируют документ для следующей сессии).
```

### Fixation strategy

Decisions made during briefing must survive compaction — if knowledge stays only in chat, it's lost. The strategy determines **where** decisions are materialized.

Choose one at index time. User approves the strategy. Record in briefing file header as `fixation: <strategy>`. The `target` field in header is mandatory for all strategies except `chat` — it names the file(s) where decisions ultimately land.

| Strategy | Where | When to use |
|----------|-------|-------------|
| `log` | In the briefing file itself — full decision + reasoning per issue | Issues are interconnected within one document/design. Fixing the target doc after each issue risks rework when a later issue changes context. All decisions first → one coherent pass on the target at the end |
| `artifact` | In a design doc or review document | Forming an intermediate artifact for another agent (or yourself in a new session) to execute. Typically: design doc for implementation, or review document for fixes |
| `final` | In code and specs directly | Issues are independent. Each can be fully closed on the spot without affecting others |
| `chat` | Nowhere — decisions stay in chat, index gets a status label only | **Anti-pattern by default.** Acceptable only for very small briefings (2-3 trivial issues) that fit in one short session with no compaction risk. If in doubt — use `log` instead. The danger: a "small" briefing grows deeper than expected, and by then the decisions are already scattered across chat with no materialization |

**Why not always `final` or `artifact`?** When issues are interconnected, fixing the target after issue 2 may need to be redone after issue 5 changes the context. `log` collects all decisions first, then applies them in one pass.

**Why not always `log`?** When issues are independent, deferring creates unnecessary indirection. Fix it, close it, move on.

**Safety net:** `!упакуй` (checkpoint) walks every chat message and verifies that all decisions made it into files. If something slipped through during fixation — checkpoint catches it.

#### Briefing file header

```markdown
# Briefing: <case name>
source: <file(s) where issues come from>
target: <file(s) where decisions land — code, spec, design doc>
fixation: <strategy>

## Issues

### 1. Some issue title
- context: <pointers>
- status: **resolved** — <one-line summary>

### 2. Another issue
- context: <pointers>
- status: pending

## Solutions

### 1. Some issue title
**Decision:** <what we decided>
**User approved:** "<exact quote>"

<reasoning — free form, as long as needed. May use #### and #####
subsections to structure complex analysis. May include alternatives
considered, arguments for and against, nuances from discussion.
One paragraph or several pages — whatever captures the depth of the
conversation so a new session doesn't repeat the work.>
```

For `final` and `artifact` — the briefing file has only the Issues section (no Solutions). Decisions go directly into `target`.

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
5. **Act** based on fixation strategy:
   - **`final`** — fix immediately in target (code, spec, design doc). Context is hot — deferring means losing it.
   - **`log`** — write full decision + reasoning in the Decisions section of the briefing file. Target document is updated in one pass after all issues are resolved.
   - **`artifact`** — write into the intermediate document (design doc or review doc).
6. **Update index** — mark status, write one-line summary of decision
7. **Zero loose ends gate** — before moving to next issue, print this checklist in chat:

   ```
   ✅ ZERO LOOSE ENDS:
   - User approved: "<exact user quote>"
   - Fixated: <where — briefing log / design doc / code+spec>
   - Index updated: <yes/no>
   - Decision recorded: <one-line summary>
   ```

   The "User approved" field is mandatory. You must quote the user's actual words that constitute approval. If you cannot fill this field — you skipped step 4. Stop, revert any changes, and go back to waiting.

   Do NOT present the next issue until this gate is printed and complete.

8. **Ask permission to move on.** After printing ZERO LOOSE ENDS — ask "Идём дальше?" and wait. This is `[review]` applied to briefing: agent cannot see the full picture and cannot mark an issue as closed — only the user can. Move to the next issue ONLY on explicit "да" or "дальше" (or user said "дальше" earlier, before the gate). Everything else means stay:
   - "Да" in response to "Согласен?" = approval of recommendation, NOT permission to move on
   - User asks a question or makes a statement ignoring "Идём дальше?" = they're ignoring the question = we're not moving on
   - User provides additional context, examples, corrections = still on current issue

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
| Mark "resolved" without materializing | A status label is not fixation. "Resolved" in index + nothing in files = lost knowledge | Fixate per chosen strategy, then mark |
