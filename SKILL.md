# llm-wiki

LLM-managed personal knowledge wiki. Создаёт и поддерживает базу знаний проекта из markdown-файлов по методике Karpathy + расширения.

---

## Использование

```
/llm-wiki              → показать это сообщение
/llm-wiki init         → развернуть структуру в текущем проекте
/llm-wiki ingest       → обработать новые файлы из raw/
/llm-wiki query        → ответить на вопрос по wiki
/llm-wiki lint         → health-check wiki
/llm-wiki end          → завершить сессию, записать резюме
```

---

## INIT — развернуть структуру

При вызове `/llm-wiki init`:

1. Спросить у пользователя: название проекта и первый slug (kebab-case ASCII ≤ 40 chars).
2. Создать структуру:

```
<project-root>/
├── raw/
│   ├── voice/
│   ├── articles/
│   ├── screenshots/
│   └── obsidian/
└── llm-wiki/
    ├── CLAUDE.md         ← только локальная специфика проекта
    ├── index.md
    ├── log.md
    ├── _meta/
    │   ├── schema.yaml
    │   └── taxonomy.md
    ├── templates/
    │   ├── project.md
    │   ├── task.md
    │   ├── reflection.md
    │   ├── note.md
    │   └── idea.md
    ├── notes/
    ├── tasks/
    ├── ideas/
    ├── reflections/
    └── entities/
```

3. Создать `llm-wiki/CLAUDE.md` с минимальным содержимым (только taxonomy + проектные исключения). Глобальные правила НЕ дублировать — они в этом скилле.

4. Скопировать шаблоны из `~/.claude/skills/llm-wiki/templates/` в `llm-wiki/templates/`.

5. Добавить в корневой `CLAUDE.md` проекта:
   ```
   ## Настройки сессии
   @llm-wiki/CLAUDE.md
   ```

6. Записать в `llm-wiki/log.md`: `[YYYY-MM-DD] init | структура создана`

Шаблоны для `llm-wiki/CLAUDE.md`:

```markdown
# llm-wiki — локальная конфигурация

## Taxonomy

| Slug | Название | Статус |
|------|----------|--------|
| `<slug>` | <Название проекта> | active |

## Специальные правила
*(если есть — добавить здесь)*
```

Шаблоны для `llm-wiki/_meta/taxonomy.md` и `llm-wiki/_meta/schema.yaml` — взять из `~/.claude/skills/llm-wiki/TAXONOMY-template.md` и `~/.claude/skills/llm-wiki/SCHEMA.yaml`.

---

## INGEST — обработать источники

При вызове `/llm-wiki ingest`:

1. Прочитать `raw_read_last_date` из `llm-wiki/_meta/schema.yaml`.
2. Найти все файлы в `raw/` с датой создания/изменения после `raw_read_last_date`.
3. Для каждого нового файла:
   - Прочитать файл.
   - Если суть неочевидна — обсудить с пользователем 3–5 ключевых takeaways.
   - Написать summary-страницу в `llm-wiki/notes/<slug>.md` с frontmatter `source: ingest`, `raw_ref: raw/...`.
   - Обновить до 15 связанных страниц (кросс-ссылки, entities, концепции).
4. Обновить `llm-wiki/index.md`.
5. Записать в `llm-wiki/log.md`: `[YYYY-MM-DD] ingest | <файл> | <slug>`
6. Обновить `raw_read_last_date` в `schema.yaml` на текущую дату.

**Запрещено**: редактировать или удалять файлы в `raw/`.

---

## QUERY — ответить на вопрос

При вызове `/llm-wiki query` или явном вопросе пользователя:

1. Прочитать `llm-wiki/index.md` — найти релевантные страницы.
2. Открыть 3–7 наиболее релевантных страниц.
3. Синтезировать ответ с цитатами формата `[title](path)`.
4. Предложить сохранить хороший ответ как новую wiki-страницу (`type: note`, `source: query-derived`).

---

## LINT — health-check

При вызове `/llm-wiki lint`:

1. Проверить все страницы на:
   - Наличие обязательного frontmatter (type, title, created, updated, source).
   - Противоречия между страницами.
   - Устаревшие утверждения (пометить `> ⚠️ Устарело:`).
   - Orphan-страницы (нет входящих ссылок в index.md или других страницах).
   - Slugs, которых нет в `taxonomy.md`.
2. Вывести список findings пользователю.
3. **Не менять автоматически** — только репортить. Изменения — после подтверждения.

---

## END — завершить сессию

При вызове `/llm-wiki end` или командах "завершить сессию" / "закончить сессию":

1. Добавить запись в `llm-wiki/reflections/session-resume.md` **сверху**:

```markdown
## [YYYY-MM-DD] <Тема сессии>

**Темы:** ...
**Файлы созданы:** ...
**Файлы изменены:** ...
**Ключевые решения/выводы:** ...
```

2. Добавить строку в `llm-wiki/log.md` снизу:
   `[YYYY-MM-DD] session-end | session-resume.md | <тема>`

3. Обновить `llm-wiki/index.md` если появились новые страницы.

4. Вывести пользователю резюме: темы + список изменённых файлов.

---

## Структура и правила

### Три слоя — жёсткое разделение

| Слой | Кто пишет | Правило |
|------|-----------|---------|
| `raw/` | Владелец | LLM **никогда** не редактирует и не удаляет |
| `llm-wiki/` | LLM | Владелец редактирует редко, только осознанно |
| `llm-wiki/templates/` | Оба | Шаблоны, не контент |

### Routing — куда класть страницу

**По умолчанию** (нет подпроектов): `llm-wiki/<тип>/<slug>.md`

**При наличии подпроекта**: `llm-wiki/projects/<slug>/<тип>/<slug>.md`

Подпроект создаётся **только по явной команде владельца**. Папка `projects/` не создаётся автоматически.

При создании подпроекта — голографическая структура: внутри `projects/<slug>/` свой `CLAUDE.md` и полная копия разделов.

Сомнения в роутинге → **спросить владельца**.

### Frontmatter (обязателен на каждой странице)

```yaml
---
type: project | task | reflection | note | idea | entity | source-summary
title: <human-readable>
created: YYYY-MM-DD
updated: YYYY-MM-DD       # обновлять при каждом редактировании
project: <slug>           # опционально
tags: []
status: open|wip|done|cancelled|parked   # обязательно для type=task
priority: P0|P1|P2|P3    # опционально
source: voice|text|manual|web|ingest|query-derived
raw_ref: raw/...          # путь к источнику если применимо
related: [slug, slug]
---
```

### Slugs

kebab-case ASCII ≤ 40 chars. Для reflections с датой: `YYYYMMDD-тема`.  
Slugs **только из `taxonomy.md`**. Новый slug → сначала добавить в taxonomy.

### Запрещено

- Редактировать или удалять файлы в `raw/`
- Удалять wiki-страницы без явного подтверждения владельца
- Создавать дубликаты (проверять `index.md` перед созданием)
- Использовать slug, которого нет в `taxonomy.md`
- Писать страницы без frontmatter
- Дублировать в `llm-wiki/CLAUDE.md` правила из этого скилла
