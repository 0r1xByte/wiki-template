# LLM Wiki — Schema

This file defines how the agent maintains this knowledge base. Read it at the start of
every session. Follow every rule precisely — consistency across sessions is what makes
this wiki useful over time.

---

## What this wiki is

A persistent, compounding knowledge base built on the LLM Wiki pattern. Raw sources
(PDFs, articles, clippings) are ingested once. The agent extracts structured knowledge and
maintains interlinked markdown files. You read it; the agent writes it.

The wiki gets richer with every source ingested. Cross-references, backlinks, and
contradictions are maintained by the agent — not the user.

---

## Definitions

### Domain
The **field of knowledge or discipline** a source, concept, or theme belongs to.
Answer to: *"What subject area is this from?"*

Domains are **broad** — the highest-level organising layer above concepts and themes.

**Valid domains (examples):**
- Psychology
- Philosophy
- Economics
- Monetary Theory
- History
- Biology
- Strategy
- Technology
- Theology
- Politics
- Systems Thinking
- Productivity
- Neuroscience
- Sociology

**Not domains (too specific — use concepts, subdomains, or tags instead):**
- Cognitive Bias → concept
- Compound Interest → concept
- Ancient Rome → entity or theme

A source can belong to multiple domains. Cross-domain connections are valuable —
flag them explicitly in concept and theme pages.

### Subdomain
A **more specific field within a domain**. Optional — only add when the source
clearly belongs to a recognised sub-discipline.

Examples:
- Domain: Psychology → Subdomain: Cognitive Psychology, Behavioural Economics
- Domain: Economics → Subdomain: Monetary Theory, Macroeconomics
- Domain: Philosophy → Subdomain: Ethics, Epistemology
- Domain: Technology → Subdomain: Artificial Intelligence, Cryptography

Subdomains are always nested under a parent domain. A source can have multiple
subdomains across multiple domains. The agent identifies subdomains from source content
and presents them for confirmation — users confirm, edit, or reject, but do not
invent them from scratch. If no recognised sub-discipline applies, leave it empty.

### Concept
A specific idea, framework, or mental model that appears in one or more sources.
Concepts have two tiers:

- **Framework concept** (top-level): a broad idea or discipline that can contain
  child concepts. Has no `parent-concept`. Examples: Penetration Testing, Cognitive Bias,
  Compound Interest.
- **Technique concept** (child): a specific method, tool, or sub-idea that belongs to
  a framework concept. Always has a `parent-concept`. Examples: Fuzzing (child of
  Penetration Testing), Anchoring Bias (child of Cognitive Bias).

Maximum nesting depth is 2 levels — technique concepts cannot have children of their own.

### Theme
An overarching thread or pattern that runs across multiple sources. Two tiers:

- **Domain theme**: pattern within a single domain. Set `cross-domain: false`.
  Examples: Human Rationality (Psychology), The Nature of Money (Economics).
- **Cross-domain theme**: pattern that emerges across 2+ domains — the most valuable
  insights in the wiki. Set `cross-domain: true`.
  Examples: Centralisation Breeds Fragility (Systems Thinking + Biology + Politics),
  Power Concentrates Without Checks (History + Politics + Economics).

Cross-domain themes must list all relevant domains in their `domains:` frontmatter field
and are surfaced first during queries.

### Entity
A person, organisation, place, movement, or event referenced across sources.
Five types:

- **person** — individual thinker, author, or actor. Example: Daniel Kahneman
- **organisation** — institution, company, or body. Example: The Federal Reserve
- **place** — geographic location with historical or conceptual significance. Example: Ancient Rome
- **movement** — intellectual or social current, not a formal org. Example: The Enlightenment, Austrian School of Economics
- **event** — a discrete historical occurrence referenced across sources. Example: The 2008 Financial Crisis, World War II

### Key Term
Technical vocabulary a reader needs defined to understand a source — the kind of
word you'd look up in a glossary, not an encyclopedia.

The distinction from Concept: a **Concept** is analytical — a framework or mental
model with implications worth exploring in prose ("Transfer Learning", "Overfitting").
A **Key Term** is definitional — vocabulary with a precise meaning that other notes
link through ("Tokenisation", "LSTM", "Self-Attention"). If removing the note would
leave other notes with unexplained jargon, it's a Key Term. If removing it would
lose an insight, it's a Concept.

Key Term notes live in `wiki/terms/` and follow the **Zettelkasten atomicity
principle**: one term, one definition, fully self-contained. A reader who has never
seen the source should be able to land on a term note and leave with a complete
understanding of that term and where to go next. No headers inside a term note —
if a header is needed, the note is trying to cover two terms.

### Source
A single ingested document — book, PDF, article, video transcript, podcast notes.
One source = one **index page** at `wiki/sources/{{Title}}.md` plus one **chapter file**
per chapter inside `wiki/sources/{{Title}}/`. Short sources without chapters (articles,
clippings) use only the index page — no subdirectory is created.

---

## File structure

```
wiki/
├── raw/
│   ├── pdfs/          ← drop PDFs here before ingesting
│   ├── clippings/     ← Obsidian Web Clipper exports
│   └── assets/        ← extracted images/diagrams (auto-populated by extract.py)
├── wiki/
│   ├── index.md       ← master catalog, updated on every ingest
│   ├── log.md         ← append-only history of all operations
│   ├── overview.md    ← high-level synthesis, updated periodically
│   ├── sources/
│   │   ├── Source Title.md            ← index: frontmatter + summary + chapter list
│   │   └── Source Title/              ← chapter files (books/PDFs only)
│   │       └── chapter-title.md
│   ├── concepts/      ← one .md per concept (analytical frameworks, mental models)
│   ├── terms/         ← one .md per key term (atomic Zettelkasten definitions)
│   ├── themes/        ← one .md per theme
│   ├── entities/      ← one .md per person/org/place
│   └── domains/       ← one .md per domain (subdomains nested inside)
├── AGENT.md           ← this file (schema)
└── .agent/
    ├── commands/      ← slash commands (one .md per command)
    └── skills/        ← skill definitions bundled with this wiki
```

---

## Frontmatter schema

Every `.md` file in `wiki/` must have YAML frontmatter. Never leave it out.

### sources/ — index file
```yaml
---
title: "Title of the source"
type: source
media: book          # book | article | pdf | video | podcast | paper
author: "Author Name"
published: YYYY
ingested: YYYY-MM-DD
tags: [tag1, tag2]
domains:
  - "[[Domains/Psychology]]"
subdomains:
  - "[[Domains/Psychology#Cognitive Psychology]]"
concepts:
  - "[[Concepts/Mental Models]]"
themes:
  - "[[Themes/Human Rationality]]"
chapters:                                            # books/PDFs only — omit for articles/clippings
  - "[[Sources/Title/chapter-title]]"
diagrams:
  - "[[raw/assets/Title-p12-img1.png]]"             # auto-populated by extract.py
code-samples:
  - page: 45
    language: python                                 # auto-populated by extract.py
raw: "[[raw/pdfs/filename.pdf]]"
---
```

### sources/ — chapter files
One per chapter, inside `wiki/sources/{{Title}}/`. Filename: `{{chapter-slug}}.md`
(no numeric prefix — chapter order is captured in the `chapter:` frontmatter integer field).

```yaml
---
title: "Chapter Title"
type: source-chapter
source: "[[Sources/Title]]"
chapter: 1                                           # chapter number (integer)
pages: "28–75"                                       # page range as string
tags: [tag1, tag2]
domains:
  - "[[Domains/Psychology]]"
subdomains:
  - "[[Domains/Psychology#Cognitive Psychology]]"
concepts:
  - "[[Concepts/Mental Models]]"
themes:
  - "[[Themes/Human Rationality]]"
diagrams:
  - "[[raw/assets/Title-p30-img1.png]]"
code-samples:
  - page: 31
    language: python
---
```

### concepts/
```yaml
---
title: "Concept Name"
type: concept
tags: [tag1, tag2]
domains:
  - "[[Domains/Psychology]]"
subdomains:
  - "[[Domains/Psychology#Cognitive Psychology]]"
sources:
  - "[[Sources/Title]]"
parent-concept: "[[Concepts/Parent Concept]]"   # omit if top-level concept
child-concepts:                                  # omit if leaf concept
  - "[[Concepts/Child Concept]]"
related-concepts:
  - "[[Concepts/Related Concept]]"
themes:
  - "[[Themes/Theme Name]]"
---
```

### themes/
```yaml
---
title: "Theme Name"
type: theme
cross-domain: false     # true if theme spans 2+ domains
tags: [tag1, tag2]
domains:
  - "[[Domains/Psychology]]"
subdomains:
  - "[[Domains/Psychology#Cognitive Psychology]]"
sources:
  - "[[Sources/Title]]"
concepts:
  - "[[Concepts/Concept Name]]"
entities:
  - "[[Entities/Entity Name]]"
---
```

### entities/
```yaml
---
title: "Entity Name"
type: entity
entity-type: person     # person | organisation | place | movement | event
tags: [tag1, tag2]
domains:
  - "[[Domains/Psychology]]"
subdomains:
  - "[[Domains/Psychology#Cognitive Psychology]]"
sources:
  - "[[Sources/Title]]"
concepts:
  - "[[Concepts/Concept Name]]"
themes:
  - "[[Themes/Theme Name]]"
related-entities:
  - "[[Entities/Related Entity]]"
---
```

### domains/
```yaml
---
title: "Domain Name"
type: domain
tags: [tag1]
subdomains:
  - Cognitive Psychology
  - Behavioural Economics
sources:
  - "[[Sources/Title]]"
concepts:
  - "[[Concepts/Concept Name]]"
themes:
  - "[[Themes/Theme Name]]"
---
```

### terms/
One file per Key Term. Filename matches the term exactly (e.g., `Tokenisation.md`,
`Self-Attention.md`). No headers inside the body — if a header feels needed, split
into two term notes.

```yaml
---
title: "Term Name"
type: term
sources:
  - "[[Sources/Title/chapter-slug]]"    # every chapter that introduces or uses this term
related-terms:
  - "[[Terms/Related Term]]"
---
```

Body: **prose only** — no headers, no bullet points. Structure: one-sentence
definition → one sentence on the mechanism → one sentence on why it matters or
what breaks without it → `See also [[Terms/X]], [[Terms/Y]]` to close.
Total: 5–8 sentences. Written to be fully self-contained.

---

## Linking rules

- Always use Obsidian `[[wikilink]]` syntax — never plain text references
- Subdomains link as anchors on the parent domain page: `[[Domains/Psychology#Cognitive Psychology]]`
- Every link must go **both ways**: if Source A links to Concept B, Concept B must
  link back to Source A
- Link format by type:
  - Sources (index) → `[[Sources/Title]]`
  - Source chapters → `[[Sources/Title/chapter-slug]]`
  - Concepts → `[[Concepts/Name]]`
  - Themes → `[[Themes/Name]]`
  - Entities → `[[Entities/Name]]`
  - Domains → `[[Domains/Name]]`
  - Subdomains → `[[Domains/Name#Subdomain Name]]`
- Chapter files link back to their source index via `source:` frontmatter; the source
  index lists all chapters in `chapters:` frontmatter — both directions are required
- Never create orphan pages — every new page must be linked from at least one
  existing page and from `wiki/index.md`
- Child concepts must link to their parent via `parent-concept`; parent concepts must
  list all children in `child-concepts`. Maximum nesting depth: 2 levels.
- Entities link to concepts and themes they are known for — not just to sources they
  appear in. If an entity is associated with a concept, that concept must backlink to
  the entity.

---

## Operations

### Ingest (triggered by /ingest or /richpdf)

**IMPORTANT: Always pause for subdomain confirmation before creating any files.**

1. For PDFs: run `.agent/skills/richpdf/extract.py`. For clippings or articles: read the file directly.
2. Identify all domains and subdomains from the source. Subdomains must be recognised
   academic or professional sub-disciplines — not topic areas, themes, or concepts.
3. Present findings to the user before creating any files:
   ```
   Domains identified:
   - Psychology → Cognitive Psychology, Behavioural Economics
   - Economics (no subdomains identified)

   Confirm, remove, or add to this list before I proceed.
   ```
4. Wait for user confirmation. Apply any edits. Then proceed.
5. Create `wiki/sources/{{Title}}.md` with full frontmatter + summary + `chapters:` list,
   including any confirmed subdomains. For books/PDFs, create one chapter file per chapter
   at `wiki/sources/{{Title}}/{{chapter-slug}}.md` (no number prefix — chapter order is
   captured in the `chapter:` frontmatter field), each with the **approved chapter file format**:

   **Chapter file format (required for every chapter):**
   - **Frontmatter:** `title`, `type: source-chapter`, `source`, `chapter` (int), `pages`,
     `domains`, `subdomains`, `concepts`, `themes`, `diagrams`, `code-samples`
   - **Navigation breadcrumb** at top: source link, previous chapter link, concept links, theme links
   - **Memorable quotes** in fenced ` ```Quote ``` ` blocks — inline near the relevant section
   - **Key idea bullet points** — medium verbose, 2–3 sentences each (the point, the mechanism,
     why it matters). Include inline `[[wikilinks]]` wherever related terms appear naturally.
   - **Diagrams embedded inline** immediately after the idea they illustrate, with a caption
   - **Consolidated code blocks** — clean, runnable examples
   - **Chapter summary section** with detailed bullet points at the end
   - **"Next chapter" link** at the very bottom

   **Short source index format (clippings, articles, HTML, plain text — no chapter subdirectory):**
   - Full frontmatter (same schema as source index, no `chapters:` field)
   - Body sections covering the source's main ideas (structure varies by source type)
   - **Summary section** — always required, even if no terms are found. Contains:
     - A prose paragraph (3–5 sentences) synthesising the source's core argument and
       its significance to the wiki
     - Inline `[[Terms/X]]` wikilinks within the prose where terms appear naturally

6. **Key Term extraction — one confirmation step per source** (applies to ALL source types):
   - **For books/PDFs:** fire immediately after writing each chapter file (one pass per chapter)
   - **For short sources** (clippings, articles, HTML, plain text): fire once, immediately
     after writing the source index file
   - Criteria (same for all source types): the term must be (a) domain-specific jargon
     not obvious from plain English, and (b) self-contained enough to deserve its own
     atomic note.
   - Present a confirmation prompt before creating any files:
     ```
     Key Terms identified in Ch1 — Chapter Title:

     New terms (will create wiki/terms/ notes):
       · Term A  · Term B  · Term C

     Existing terms (will add this chapter/source to sources frontmatter):
       · Term D  · Term E

     Proposed related-term connections:
       · Term A ↔ Term B

     Confirm, remove, or add before I proceed.
     ```
   - On confirmation:
     - **New terms:** create term notes in `wiki/terms/`
     - **Existing terms:** append the confirmed chapter or source to `sources:` frontmatter
       only — body is never rewritten automatically
7. **Add inline term wikilinks** — fires after all term notes for the source exist:
   - Add `[[Terms/X]]` at the **first occurrence** of each confirmed term throughout
     the file (both body sections and summary). Only link the first occurrence per file.
   - Never collect terms in a separate list — all links must be embedded in existing prose.
8. Read `wiki/index.md` to find all existing domain, concept, theme, and entity pages
9. For each **domain** identified:
   - If `wiki/domains/{{Domain}}.md` exists — open it, add source to `sources:` frontmatter,
     add any new subdomains to `subdomains:`, and add a backlink in body
   - If it does not exist — create it with full frontmatter including subdomains
10. For each **concept** identified: same as above in `wiki/concepts/`
11. For each **theme** identified: same as above in `wiki/themes/`
12. For each **entity** (person, org, place) referenced: same as above in `wiki/entities/`
13. Update `wiki/index.md` — add all new pages created, including new terms under `## Terms`
14. Append to `wiki/log.md`:
    ```
    ## [YYYY-MM-DD] ingest | {{Title}}
    Domains: A, B | Subdomains: A#X, B#Y
    Pages touched: sources/X, domains/A, concepts/B, themes/C
    ```
15. Check whether `overview.md` needs a rebuild:
    - Count `ingest` entries in `wiki/log.md` since the last `overview rebuild` entry
    - If count ≥ 3, or if a new `cross-domain: true` theme was created, prompt the user:
      ```
      overview.md rebuild recommended — N ingests since last update. Rebuild now?
      ```

### Rebuild Overview (triggered by /rebuild-overview)

1. Read `wiki/index.md` to get the full list of domains, themes, concepts, and entities
2. Read all `cross-domain: true` theme pages
3. Read all domain pages
4. Read entity pages where the entity spans 2+ domains
5. Regenerate `wiki/overview.md` following the overview.md format exactly
6. Append to `wiki/log.md`: `## [YYYY-MM-DD] overview rebuild | Triggered by: manual`

### Query

1. Read `wiki/index.md` first to identify relevant pages
2. Read `cross-domain: true` themes first — surface at the top of the answer
3. Read remaining relevant pages and synthesise a complete answer
4. If the answer is a useful synthesis — offer to file it as a new concept or theme page

### Lint (triggered by /lint)

Periodic health check. Look for:
- Orphan pages (no inbound links)
- Missing backlinks (A references B but B does not reference A) — all link types:
  source↔concept, source↔theme, source↔entity, concept↔theme, entity↔concept,
  entity↔theme, concept↔domain, theme↔domain
- Concepts or themes mentioned in source pages that lack their own wiki page
- Contradictions between pages from different sources
- Subdomains listed in source frontmatter but missing from the parent domain page
- Themes with 2+ domains in `domains:` but `cross-domain: false` — flag for promotion
- Themes with `cross-domain: true` but only one domain listed — flag as inconsistent
- **Concept parent-child integrity:**
  - Concept has `parent-concept` but the parent's `child-concepts` does not list it back
  - Concept has `child-concepts` entries that do not link back via `parent-concept`
  - Depth-2 concept that itself has `child-concepts` — violates max-depth-2 rule
- **Term integrity:**
  - Term note body contains headers (violates atomicity rule)
  - Term note body exceeds ~10 sentences (likely covering two terms — flag for split)
  - `[[Terms/X]]` wikilink appears more than once in the same file (only first occurrence should be linked)

Report findings as a list. Ask user which to fix before making changes.

---

## index.md format

```markdown
# Wiki Index

> Master catalog of all pages. Updated by the agent on every ingest.
> Read this first before any ingest or query operation.

---

## Sources
- [[Sources/Title]] — one-line description (YYYY, Author)

## Concepts
- [[Concepts/Name]] — one-line description
  - [[Concepts/Child Name]] — one-line description (child of Name)

## Terms
- [[Terms/Name]] — one-sentence definition

## Themes
- [[Themes/Name]] — one-line description

## Entities
- [[Entities/Name]] — one-line description (person) | concepts: Name, Name
- [[Entities/Name]] — one-line description (organisation)

## Domains
- [[Domains/Name]] — one-line description
  - Subdomain: Name, Name
```

---

## overview.md format

```markdown
# Wiki Overview

## Cross-Domain Themes
> Patterns that hold across multiple fields — highest-confidence insights in the wiki.

- **[[Themes/Name]]** _(Domains: A, B, C)_ — one-line summary of the pattern

## Domains
### [[Domains/Name]]
One paragraph: what this domain covers, its key framework concepts, and how it
connects to other domains.

Key concepts: [[Concepts/Name]], [[Concepts/Name]]
Key themes: [[Themes/Name]]

## Notable Entities
- **[[Entities/Name]]** _(person)_ — one-line description, domains they span
```

---

## log.md format

Append-only. One entry per operation. Never edit past entries.

```markdown
## [YYYY-MM-DD] ingest | Source Title
Domains: Psychology | Subdomains: Psychology#Cognitive Psychology
Pages touched: sources/X, domains/A, concepts/B C, themes/D

## [YYYY-MM-DD] query | Question asked
Answer filed as: concepts/New Concept (or "not filed")

## [YYYY-MM-DD] lint
Issues found: N total — X orphans, Y backlinks, Z concept violations, W cross-domain.
Fixed: #1, #3 | Deferred: #2, #4
```

---

## Rules the agent must always follow

1. **Never modify files in `raw/`** — they are immutable source of truth
2. **Always read `wiki/index.md` first** before any ingest or query
3. **Always pause and confirm subdomains with the user before creating any files**
4. **Always create backlinks in both directions** — no one-way references
5. **Always update `wiki/index.md` and `wiki/log.md`** after every ingest
6. **Never create a page without frontmatter** — schema compliance is non-negotiable
7. **One source = one index page** in `wiki/sources/` plus one chapter `.md` per chapter —
   never merge sources, never skip the index
8. **Domains are broad fields** — if in doubt, check the Definitions section above
9. **Always identify subdomains from source content** — present findings for confirmation.
   Never ask the user to invent them. If none applies, leave subdomains empty.
10. **Flag contradictions explicitly** — note in both pages rather than silently overwriting
11. **Assets stay inline with their idea** — never grouped separately or orphaned
12. **Never re-describe an asset that already has a description** — reuse existing

---

## Agent skills

### Available commands

| Command | Purpose |
|---|---|
| `/ingest` or `/richpdf` | Ingest any source type into the wiki |
| `/grill-me` | One-question-at-a-time interview to resolve a design or schema decision |
| `/to-prd` | Turn conversation context into a PRD saved to `wiki/llm_conversations/` |
| `/edit-article` | Clean up a clipping before ingesting |
| `/caveman` | Ultra-compressed mode (~75% fewer tokens) |
| `/write-a-skill` | Create a new wiki-specific skill |
| `/lint` | **Final pipeline step.** Full health check: schema, backlinks, index, concept/theme/term integrity, pipeline completion audit |
| `/rebuild-overview` | Regenerate `wiki/overview.md` |

### Issue tracker

PRDs and conversation records live as local markdown files under `wiki/llm_conversations/`.
Filename convention: `YYYY-MM-DD-<feature-slug>.md`
