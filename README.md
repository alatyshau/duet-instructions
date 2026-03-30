# Duet Instructions

Пользовательский воркспейс AI-инструкций для платформы [Duet](https://github.com/alatyshau/duet).

## Что это

Персональная база инструкций для AI-агентов: персоны, скиллы, core-правила, схемы, скрипты.

Duet Host подключает этот воркспейс через конфигурацию (`instructionsPath` в `DuetConfig/{machine}.json`). Backend читает `index.json` для построения каталога, а `core_instructions.md` компонуется с платформенным bootstrapper'ом в единый output-style для AI-клиентов.

## Двуслойная архитектура

| Слой | Где | Что | Кто владеет |
|------|-----|-----|-------------|
| **Bootstrapper** | `packages/backend/bootstrapper.md` (в Duet) | Ориентация, глоссарий, three roots | Duet (платформа) |
| **Core instructions** | `core_instructions.md` (здесь) | Правила (L7+, honesty, safe, review), observable rules, spec-driven | Пользователь |

Backend компонует оба слоя через маркер `<!-- INSERT USER CORE INSTRUCTIONS -->` → готовый merged output для AI-клиентов.

## Структура

```
index.json                  ← декларация структуры (core_instructions, personas, skill_folders)
core_instructions.md        ← пользовательские правила для AI-агентов
personas/                   ← WHO — идентичности агента на сессию
  socrates.md
  hephaestus.md
  ...
skills/                     ← WHAT — знания и умения, загружаются по запросу
  coding/                   ←   языки программирования
  modes/                    ←   режимы работы (planning, execute, review...)
  stances/                  ←   подходы к мышлению (dialectic, pragmatic...)
  tools/                    ←   доменные инструменты (spec-architect, scriptor...)
  workflows/                ←   паттерны сотрудничества (solo, pair, sddg)
schemas/                    ← форматы файлов (topic_file, index, skill_file)
scripts/                    ← Python-код для сложных скиллов
```

## index.json

Декларирует структуру воркспейса. Backend использует его для построения каталога в `workspace_info` и для нахождения `core_instructions.md`.

```json
{
  "core_instructions": "core_instructions.md",
  "personas": { "path": "personas" },
  "skill_folders": [
    { "name": "Coding", "path": "skills/coding" },
    { "name": "Modes", "path": "skills/modes" }
  ]
}
```

## Система концепций

| Концепция | Вопрос | Длительность |
|-----------|--------|-------------|
| **Persona** | КТО я? | Вся сессия |
| **Skill** | ЧТО я знаю / умею? | По запросу |

Modes, stances, workflows — категории скиллов.

## YAML frontmatter

Каждый `.md` файл персоны/скилла содержит YAML frontmatter:

```yaml
---
name: socrates
description: Research, dialectics
shortcuts: ["Сократ"]
---
```

- **Обязательные:** `name`, `description`
- **Опциональные:** `shortcuts` (список), `trigger`, `noTrigger` (только скиллы)
