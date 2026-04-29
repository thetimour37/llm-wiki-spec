# llm-wiki

> LLM-managed personal knowledge wiki for any project.

A skill for [Claude Code](https://claude.ai/code) that creates and maintains a structured knowledge base inside your project. Based on [Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) with extensions.

---

## What it does

- **Builds** a `llm-wiki/` knowledge base in your project with one command
- **Ingests** files from `raw/` and compiles them into structured wiki pages
- **Answers** questions by reading wiki pages and synthesizing responses
- **Maintains** cross-links, detects contradictions, finds orphan pages
- **Tracks** sessions with automatic summaries

The key idea: knowledge accumulates **once** in `.md` files and gets updated — not re-derived on every query. This is **not RAG**.

---

## Install

### Global (recommended)

```bash
# Clone the spec repo
git clone https://github.com/<your-username>/llm-wiki-spec ~/projects/llm-wiki-spec

# Symlink into Claude Code skills directory
mkdir -p ~/.claude/skills
ln -s ~/projects/llm-wiki-spec ~/.claude/skills/llm-wiki
```

**Windows (Developer Mode or Admin terminal):**
```powershell
git clone https://github.com/<your-username>/llm-wiki-spec "$env:USERPROFILE\projects\llm-wiki-spec"
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills"
New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\llm-wiki" -Target "$env:USERPROFILE\projects\llm-wiki-spec"
```

### Project-level (no symlink needed)

```bash
cp -r llm-wiki-spec .claude/skills/llm-wiki
```

---

## Usage

In any Claude Code session:

```
/llm-wiki init     → scaffold llm-wiki structure in current project
/llm-wiki ingest   → process new files from raw/
/llm-wiki query    → answer a question from wiki
/llm-wiki lint     → health-check (orphans, contradictions, missing frontmatter)
/llm-wiki end      → close session with summary written to session-resume.md
```

---

## Project structure after init

```
your-project/
├── raw/                    ← your source files (IMMUTABLE, LLM never edits)
│   ├── voice/              (transcripts)
│   ├── articles/           (articles, PDFs)
│   ├── screenshots/
│   └── obsidian/           (Obsidian vault mirror, optional)
└── llm-wiki/               ← LLM writes here
    ├── CLAUDE.md           (local taxonomy + project-specific rules only)
    ├── index.md            (page catalog)
    ├── log.md              (append-only operation log)
    ├── _meta/
    │   ├── schema.yaml     (frontmatter spec + raw_read_last_date)
    │   └── taxonomy.md     (project slugs + tags registry)
    ├── templates/          (page templates)
    ├── notes/
    ├── tasks/
    ├── ideas/
    ├── reflections/        (includes session-resume.md)
    └── entities/
```

Subprojects (holographic): when a subproject is explicitly created, `llm-wiki/projects/<slug>/` gets its own full structure.

---

## Spec files in this repo

| File | Purpose |
|------|---------|
| `SKILL.md` | Claude instructions — loaded when `/llm-wiki` is called |
| `BOOTSTRAP.md` | Full methodology reference |
| `SCHEMA.yaml` | Canonical frontmatter spec (copied to each project) |
| `TAXONOMY-template.md` | Template for per-project taxonomy |
| `templates/` | Page templates (note, task, reflection, idea, project) |
| `README.md` | This file |

---

## Updating the spec

```bash
cd ~/projects/llm-wiki-spec
git pull
```

All projects using the symlink pick up changes immediately on next session — no per-project updates needed.

---

## License

MIT
