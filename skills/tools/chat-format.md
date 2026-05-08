---
name: chat-format
description: Format an exported AI chat (Prompt/Response markdown) into Q-numbered sections with brief titles for a navigable TOC
shortcuts: ["!чат-формат", "!chat-format"]
trigger: "User has an exported AI chat file (Gemini, ChatGPT, Claude, etc.) with repeated `## Prompt:` / `## Response:` markers and wants navigable structure. Phrases like 'отформатируй чат', 'сделай оглавление из чата', 'format this chat export', 'красивый формат для чата'."
noTrigger: "User wants PDF/HTML output — that's `!рендер`, run after this skill. User wants to summarize, condense, or rewrite content — chat-format only restructures markers, never edits the body text."
---
# Skill: Chat-Format

Turn a flat exported chat file into a navigable document with numbered Q-sections and meaningful titles.

## Why

A raw export from Gemini/ChatGPT/Claude is a linear stream of `## Prompt:` / `## Response:` headings. Twenty turns in, the sidebar TOC is just "Prompt, Response, Prompt, Response..." — useless. The user can't jump to "where did we discuss SAP migration?" without scrolling.

The fix is structural:

- Each Prompt/Response pair collapses into one H2: `## Q01 — Brief title`. The TOC reads like a real table of contents.
- The user's message and the assistant's reply are separated inside the section by a horizontal rule. Visually distinct turns, no decorative labels needed.
- Subheadings inside the answer stay where they were (`###`, `####`) — they nest naturally under the Q-heading.

After this, `!рендер` gives a PDF with proper bookmarks and a clickable sidebar by topic.

## Quality Criteria

- **Titles are concrete and specific.** "Миграция Google с Oracle на SAP" — not "Вопрос про SAP" or "Корпоративный софт". A reader scanning the TOC should recognize each turn.
- **Titles match what the USER asked**, not what the assistant covered. The user navigates their own chat by how they remember the question, not by the answer's structure.
- **Length: 2-6 words.** Long enough to disambiguate, short enough to scan in a sidebar.
- **All pairs numbered, none skipped.** Even one-liner exchanges get their own Q-heading.
- **Body text untouched.** Only the headings change. Never rewrite, paraphrase, or re-flow prompt/response content.

## Pipeline

### 1. Detect the markers

Different sources use different markers. Inspect the file before writing the script:

```bash
grep -nE "^(## Prompt:|## Response:|### You:|### Assistant:|\*\*You:\*\*|\*\*ChatGPT:\*\*)" file.md | head -20
```

Common patterns:

- Gemini export: `## Prompt:` / `## Response:`
- ChatGPT markdown export: often `### You:` / `### ChatGPT:` or bold variants
- Claude.ai export: varies — inspect first

The substitution logic stays the same; only the literal markers change between sources.

### 2. Read every pair, draft titles

Read the file in full. For each Prompt/Response pair, write down a 2-6 word title capturing what the user asked. Verify the title against the question: would the user, scanning their own TOC, recognize this turn? If the title could fit any of three different turns, it's too generic — sharpen it.

### 3. Run a Python script

Python's `re.sub` with a callback handles per-occurrence numbering cleanly. Sed/awk struggle with Unicode titles + counters.

```python
import re

PATH = "<absolute path to chat file>"
TITLES = [
    "Title for Q01",
    "Title for Q02",
    # ... one entry per Prompt/Response pair, in order
]

# Adjust these two literals if the file uses different markers
PROMPT_MARKER = "## Prompt:\n"
RESPONSE_MARKER = "## Response:\n"

with open(PATH, "r", encoding="utf-8") as f:
    content = f.read()

n_prompt = content.count(PROMPT_MARKER)
n_response = content.count(RESPONSE_MARKER)
assert n_prompt == len(TITLES), f"Found {n_prompt} prompts, expected {len(TITLES)}"
assert n_response == len(TITLES), f"Found {n_response} responses, expected {len(TITLES)}"

counter = {"i": 0}
def replace_prompt(_m):
    i = counter["i"]
    counter["i"] += 1
    return f"## Q{i+1:02d} — {TITLES[i]}\n\n"

content = re.sub(re.escape(PROMPT_MARKER), replace_prompt, content)
content = content.replace(RESPONSE_MARKER, "---\n\n")

with open(PATH, "w", encoding="utf-8") as f:
    f.write(content)

print(f"Done. Replaced {len(TITLES)} prompt/response pairs.")
```

The assertion is load-bearing: if marker counts and title count disagree, the file has a malformed pair (orphan marker, stray content). Fix the file or the title list before running.

### 4. Verify

```bash
grep -nE "^## " file.md
```

Visual sanity check that H2 headings are now `Q01 — ...`, `Q02 — ...`, sequential, with sensible titles. Past 99 pairs, switch to `Q001`-style padding.

## Format conventions

- `## Q01 — Title` — em dash (`—`, U+2014), zero-padded to two digits.
- Horizontal rule is `---` on its own line, with blank lines above and below. Two blank lines around it prevents accidental setext-heading interpretation in CommonMark.
- Subheadings inside answers keep their original level. Do not promote `###` to `##` — that would shadow the Q-numbering and break the TOC.

## Anti-patterns

| Don't | Why |
|-------|-----|
| Generate titles without reading the body | "Вопрос N", "Уточнение", "Продолжение" defeat the entire point — the TOC is the deliverable |
| Title from the answer instead of the question | The user navigates by what they asked. "Three levels of languages" feels disconnected when the actual prompt was "Правильно ли я понимаю что у нас выходит три уровня языков?" |
| Use sed/awk for the substitution | Per-occurrence counter with Unicode titles → Python `re.sub(callback)` is the clean path |
| Skip the horizontal rule between prompt and response | Without the visual break, where the user message ends and the assistant reply begins becomes a guess |
| Write `## Q01:` or `## Q01.` instead of `## Q01 — ` | Em dash reads naturally in both Russian and English; consistency across docs |
| Edit prompt or response text | Out of scope. Restructuring markers is not an editorial pass. If the user wants the chat condensed, that's a different task |
| Drop the assertion in the script | Silently produces partial output if title list and marker count disagree. Assertion fails fast and points at which side is wrong |
| Render to PDF inside this skill | Out of scope — `!рендер` is a separate, composable step. Don't bundle |
