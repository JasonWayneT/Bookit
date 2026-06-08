# Bookit — Book System

> An AI pipeline that transforms raw expert content (transcripts, notes, discussions) into structured, beginner-accessible digital books formatted as visual reports.

---

## What This Is

Subject matter experts produce valuable content in formats that are inaccessible to beginners — meeting transcripts, fragmentary notes, verbal explanations that assume existing knowledge. Bookit is an AI-first pipeline that takes that raw material and produces publication-ready digital books with clear structure, beginner scaffolding, visual diagrams, and validated pedagogy.

**What this is not:**
- A word processor or manual writing tool — the pipeline does the heavy lifting
- A fiction writing system — designed for non-fiction, educational, and expert content only
- A finished product — currently in prototype/design phase with no public release

---

## Status

| Field | Value |
|---|---|
| **Phase** | Prototype |
| **Stability** | Design-stage — no public release |
| **Last updated** | May 2026 |

---

## How It Works

1. **Ingest** — raw content in (transcripts, notes, slides, expert discussions)
2. **Classify & structure** — AI identifies topics, headings, and content hierarchy
3. **Expand** — explanations rewritten for beginners; assumed knowledge surfaced and defined
4. **Enrich** — visual elements (diagrams, callouts, charts) generated from content
5. **Validate** — pedagogy rules and quality checklists run against output
6. **Output** — publication-ready vertical visual report

---

## What's In This Repo

This repo contains the content architecture and production specifications:

| File | Purpose |
|---|---|
| `BOOK_ARCHITECTURE.md` | Overall book structure and hierarchy rules |
| `BOOK_STYLE_MANIFEST.md` | Writing style and tone guidelines |
| `BOOK_TAXONOMY.md` | Content categorization and tagging system |
| `CHAPTER_TEMPLATE.md` | Template for individual chapter structure |
| `CONTENT_MODELS.md` | Data models for different content types |
| `PEDAGOGY_RULES.md` | Teaching principles applied to content |
| `TRANSFORMATION_RULES.md` | Rules for converting raw input to structured output |
| `VALIDATION_CHECKLISTS.md` | Quality gates before publication |
| `VISUAL_CONTENT_MODEL.md` | Structure for diagrams, callouts, and visual elements |
| `VOCABULARY_AND_GLOSSARY_MODEL.md` | Terminology management and glossary generation |
| `product-brief.md` | Full product brief and requirements |
| `prd.md` | Product Requirements Document |

---

## How This Was Built

The architecture was defined before any code. Product brief and PRD were written first to clarify the scope and constraints, then content models and transformation rules were designed to govern how the AI pipeline would behave. The spec-first approach matters here because the output quality depends on the rules being precise — vague rules produce vague books.

---

## License

Private
