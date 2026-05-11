---
name: richpdf
description: >
  Extract structured wiki-ready markdown from any document — PDF (text or image/scanned),
  markdown clippings, or plain text. Captures text, embedded images/diagrams, and code
  samples per page. Images saved to raw/assets/ and referenced inline with the page they
  came from. Use when user says "/richpdf", "ingest", "process this pdf", "ingest this document",
  or provides a file path.
---

# richpdf / ingest

Extracts structured, wiki-ready markdown from any document. Per page it captures:
- 5 key ideas
- All embedded images and diagrams (saved to `raw/assets/`, described by vision)
- All code samples (detected by font analysis or fenced blocks, language identified)

Each image and code sample is placed inline immediately after the key ideas of the page
it came from — so the extracted knowledge stays in context.

## Requirements (PDF extraction)

```bash
pip install pymupdf anthropic
```

## Usage (PDFs)

```bash
python .agent/skills/richpdf/extract.py <file_path> [output_dir]
```

- `file_path` — path to document (PDF, .md, .txt)
- `output_dir` — where to save the wiki `.md` (default: `wiki/sources/`)
- Images always saved to `raw/assets/`

For clippings and articles: read the file directly — no script needed.

## How assets are detected

| Source type | Text | Code blocks | Images/Diagrams |
|-------------|------|-------------|-----------------|
| PDF (text-based) | pymupdf text extraction | Font analysis — monospace spans flagged as code | pymupdf image extraction per page |
| PDF (image/scanned) | Vision reads whole page | Vision wraps code in ``` fences | Whole page sent to vision; individual images also extracted |
| Markdown clipping | Direct read | ``` fenced blocks | `![[wikilink]]` and `![alt](url)` references captured |
| Plain text | Direct read | ``` fenced blocks | n/a |

Images under 5KB (icons, bullets, decorations) are skipped automatically.

## Asset file naming

```
raw/assets/{Source-Title}-p{page}-img{index}.{ext}
```

## After extraction

Follow all steps in the **Ingest** operation defined in `AGENT.md`.
