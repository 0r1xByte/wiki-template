---
name: lint
description: >
  Full wiki health check and pipeline completion audit. Verifies every requirement
  in AGENT.md: schema compliance, bidirectional backlinks, orphan pages,
  concept/theme/term integrity, index completeness, and ingest pipeline completion.
  Always invoke as the final step after any ingest. Use when user says "/lint",
  "check the wiki", "run lint", or "verify the pipeline".
---

# Wiki Lint

Full health check and pipeline completion audit. Run after every ingest as the final
pipeline step. Scans all files in `wiki/`, cross-references against `wiki/index.md`
and `wiki/log.md`, and reports every violation before making any change.

## Process

### Step 1 — Load the state

Read in parallel:
- `wiki/index.md` — master catalog
- `wiki/log.md` — operation history
- All files in `wiki/sources/`, `wiki/concepts/`, `wiki/terms/`, `wiki/themes/`,
  `wiki/entities/`, `wiki/domains/`

Build an in-memory map: `{ path → { frontmatter, body, outbound_wikilinks } }`

---

### Step 2 — Run all checks

Collect every violation as a numbered issue `[#N]` with file path and description.
Group by category. Do NOT fix anything yet.

---

#### A — Schema compliance

For every `.md` file in `wiki/`:

1. **Missing frontmatter** — file has no YAML `---` block
2. **Missing `type` field** — frontmatter exists but `type:` is absent
3. **Wrong type value** — `type:` is not one of `source`, `source-chapter`, `concept`,
   `theme`, `entity`, `domain`, `term`
4. **Required fields absent** — check per type:
   - `source` → `title`, `media`, `author`, `ingested`, `domains`
   - `source-chapter` → `title`, `source`, `chapter`, `pages`, `domains`
   - `concept` → `title`, `domains`, `sources`
   - `theme` → `title`, `cross-domain`, `domains`, `sources`
   - `entity` → `title`, `entity-type`, `domains`, `sources`
   - `domain` → `title`, `sources`
   - `term` → `title`, `sources`, `related-terms`
5. **Invalid `media` value** — not one of `book`, `article`, `pdf`, `video`, `podcast`, `paper`
6. **Invalid `entity-type` value** — not one of `person`, `organisation`, `place`, `movement`, `event`
7. **Short source has `chapters:` field** — clippings/articles must omit `chapters:`
8. **Book/PDF source missing `chapters:` field** — media `book` or `pdf` must have `chapters:`

---

#### B — Index completeness

9. **Page missing from index** — file exists in `wiki/` but has no entry in `wiki/index.md`
10. **Index entry points to missing file** — `wiki/index.md` references a page that does
    not exist on disk
11. **Index section missing** — `wiki/index.md` is missing one or more of:
    `## Sources`, `## Concepts`, `## Terms`, `## Themes`, `## Entities`, `## Domains`
12. **New terms not in index** — `wiki/terms/` contains a file with no entry under
    `## Terms` in `wiki/index.md`

---

#### C — Backlink integrity (bidirectional)

13. **source → concept missing reverse**
14. **source → theme missing reverse**
15. **source → entity missing reverse**
16. **concept → theme missing reverse**
17. **entity → concept missing reverse**
18. **entity → theme missing reverse**
19. **concept → domain missing reverse**
20. **theme → domain missing reverse**
21. **chapter → source index missing reverse**
22. **source index → chapter missing reverse**
23. **Orphan page** — no inbound links from any page AND not in `wiki/index.md`

---

#### D — Subdomain integrity

24. **Subdomain in source but missing from domain page**
25. **Domain page lists subdomain not used by any source** — flag for review

---

#### E — Concept integrity

26. **Parent-child asymmetry (parent missing child)**
27. **Parent-child asymmetry (child missing parent)**
28. **Depth-3 violation** — concept has both `parent-concept` and `child-concepts`
29. **Concept wikilink in source but page doesn't exist**

---

#### F — Theme integrity

30. **cross-domain: false but 2+ domains listed** — flag for promotion
31. **cross-domain: true but only one domain** — inconsistent
32. **Theme wikilink in source but page doesn't exist**

---

#### G — Term integrity

33. **Term body contains headers** — violates atomicity rule
34. **Term body too long** — exceeds 10 sentences, likely covers two terms
35. **Term `sources:` entry missing**
36. **Term source points to non-existent file**
37. **Term not backlinked in source** — term's sources list a file but `[[Terms/X]]`
    doesn't appear in that file
38. **Duplicate term wikilink in same file** — only first occurrence should be linked
39. **Short source missing Summary section**

---

#### H — Pipeline completeness

For each `ingest` entry in `wiki/log.md`:

40. **Source file missing** — log records ingest but source .md doesn't exist
41. **Domain pages not created**
42. **Concept pages not created**
43. **Theme pages not created**
44. **Entity pages not created**
45. **Source not in index**
46. **Term extraction possibly skipped** — technical source with no term wikilinks and no
    terms listing it in `sources:`
47. **Chapter files missing** — `chapters:` lists files that don't exist on disk
48. **Overview stale** — 3+ ingests since last `overview rebuild` in log

---

#### I — Log completeness

49. **Source exists but no ingest log entry**
50. **Log entries out of chronological order** — possible edit to append-only log

---

### Step 3 — Report

```
Wiki Lint Results — YYYY-MM-DD
══════════════════════════════

A — Schema compliance         [N issues]
  [#1] wiki/concepts/Foo.md — missing required field: sources

B — Index completeness        [N issues]
  [#2] wiki/terms/Bar.md — exists on disk but missing from wiki/index.md

...

H — Pipeline completeness     [N issues]
  [#N] ingest logged for "Title" but wiki/sources/Title.md missing

Total: N issues across N categories.
══════════════════════════════
```

If zero issues: print `✓ Wiki is clean. All pipeline requirements satisfied.` and stop.

---

### Step 4 — Confirm fixes

```
Which issues would you like me to fix?
Options: "all", "all except #N,#M", specific numbers, or "none".
```

Wait for response before making any changes.

---

### Step 5 — Fix confirmed issues

Minimal fixes per issue type:
- **Missing backlink** → add wikilink to appropriate frontmatter field or body
- **Missing index entry** → add correct line under right section in `wiki/index.md`
- **Missing frontmatter field** → add field with placeholder, note it needs filling
- **Missing page** → create stub with correct frontmatter and placeholder body
- **Orphan page** → link from most appropriate existing page AND add to index
- **cross-domain flag mismatch** → update flag in theme frontmatter
- **Duplicate term wikilink** → remove all but first occurrence
- **Missing Summary section** → add empty `## Summary` with placeholder paragraph
- **Overview stale** → prompt user to rebuild

Issues requiring judgment (contradictions, term splits, schema design) → flag for human review, do not auto-fix.

---

### Step 6 — Log the lint run

Append to `wiki/log.md`:

```
## [YYYY-MM-DD] lint
Issues found: N total — A schema, B index, C backlinks, D subdomains, E concepts,
F themes, G terms, H pipeline, I log.
Fixed: #1, #3 | Deferred: #2, #5
```
