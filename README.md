# LLM Wiki Template

A generic, agent-agnostic starter for the LLM Wiki pattern — a persistent, compounding
knowledge base where raw sources (PDFs, articles, clippings) are ingested once and
Claude extracts structured, interlinked knowledge.

---

## Directory structure

```
wiki-template/
├── raw/
│   ├── pdfs/          ← drop PDFs here before ingesting
│   ├── clippings/     ← web clipper exports / markdown articles
│   └── assets/        ← extracted images/diagrams (auto-populated)
├── wiki/
│   ├── index.md       ← master catalog
│   ├── log.md         ← append-only operation history
│   ├── sources/       ← one index page + chapter subdirectory per source
│   ├── concepts/      ← analytical frameworks and mental models
│   ├── terms/         ← atomic Zettelkasten key-term definitions
│   ├── themes/        ← cross-source patterns and threads
│   ├── entities/      ← persons, organisations, places, movements, events
│   └── domains/       ← top-level fields of knowledge
├── .agent/
│   ├── commands/      ← slash command stubs (see § Slash commands below)
│   └── skills/        ← SKILL.md instruction files for each command
└── AGENT.md           ← full schema and operating rules (read this first)
```

---

## Getting started

1. Copy this template directory into your project root.
2. Read `AGENT.md` — it defines every rule Claude must follow.
3. Drop a PDF into `raw/pdfs/` or a clipping into `raw/clippings/`.
4. Run `/ingest` (or `/richpdf` for PDFs) to start the pipeline.
5. Run `/lint` after every ingest to verify pipeline completeness.

---

## Slash commands

Commands live in `.agent/commands/`. Each file is a one-liner that tells the agent
which `SKILL.md` to read and follow.

### Runtime configuration

The `.agent/commands/` path is the **template convention** — agent-agnostic and safe
to commit. Whether your agent runtime reads from this path natively depends on the
tool you're using:

| Runtime | Native command path | How to wire up `.agent/commands/` |
|---|---|---|
| **Claude Code** (CLI / IDE extension) | `.claude/commands/` | Copy or symlink `.agent/commands/` → `.claude/commands/`, or copy individual files |
| **Cursor** | `.cursor/rules/` or project rules UI | Copy command stubs into Cursor's rules directory |
| **Windsurf** | `.windsurf/rules/` | Copy command stubs into Windsurf's rules directory |
| **Other agents** | Varies | Point your agent at `.agent/commands/<name>.md` manually, or copy to the runtime's expected path |

**Claude Code quick setup** — run once from the repo root:

```bash
# Option A: symlink (changes to .agent/commands/ reflect instantly)
mkdir -p .claude && ln -s ../.agent/commands .claude/commands

# Option B: copy (independent copies, update manually)
cp -r .agent/commands .claude/
```

After copying or symlinking, Claude Code will register each file as a `/command-name`
slash command automatically.

> **Why two directories?** `.claude/commands/` is Claude Code's native path and is
> often gitignored or user-local. `.agent/commands/` is committed alongside the skills
> so the commands travel with the project regardless of which agent reads them.
> The indirection is intentional — one source of truth, multiple consumers.

---

## Available commands

| Command | What it does |
|---|---|
| `/ingest` | Full wiki ingest for any source type (PDF, article, clipping) |
| `/richpdf` | Same as `/ingest`, optimised for PDFs with extract.py |
| `/lint` | Full health check — run after every ingest |
| `/grill-me` | Socratic Q&A to design or refine a spec |
| `/to-prd` | File a conversation as a PRD in `wiki/llm_conversations/` |
| `/edit-article` | Edit and improve prose in any markdown article |
| `/caveman` | Explain a concept in plain language |
| `/write-a-skill` | Scaffold a new SKILL.md for a new command |
