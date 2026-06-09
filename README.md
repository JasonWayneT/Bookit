---
title: "Bookit"
description: "An AI pipeline for transforming raw expert content — transcripts, notes, discussions — into publication-ready digital books with beginner-accessible pedagogy baked in."
author: "Jason Taylor"
role: "Product Manager"
status: "in-progress"
ai_role: "content pipeline execution following Jason's transformation rules; visual generation within Jason's art direction specs"
tech_stack: ["Markdown", "AI Pipeline", "Prompt Engineering"]
pm_skills: ["information architecture", "content strategy", "pedagogy design", "spec-first pipeline design", "product brief", "transformation rules design"]
keywords: ["content transformation", "AI pipeline", "digital books", "pedagogy", "expert content", "publication", "knowledge product"]
date_completed: ""
---

# Bookit — Book System

> An AI pipeline for turning raw expert content into publication-ready digital books. Transcripts, notes, or discussions in; structured, beginner-accessible visual report out.

---

## What This Is

Experts produce valuable content in formats beginners can't use. Meeting transcripts, fragmented notes, verbal explanations full of assumed knowledge. Getting from that raw material to something teachable requires editorial work most experts don't have time for. Bookit does that work.

Three things distinguish it from a reformatter. Every output element traces back to source material: the AI restructures and clarifies, it doesn't invent. Pedagogy rules are baked into the transformation, not added after. Validation runs automatically before any output is considered done.

**My role:** Architecture design, all content models and transformation rules, pedagogy rules, product brief, art direction specs, validation checklists.
**AI's role:** Content pipeline execution following my transformation rules, visual generation within my art direction. The rules govern what AI does; AI doesn't author the rules.

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

## Results & Impact

- **Prototype output:** Visual report generated from Applyr source content — demonstrated the full pipeline from raw expert material to structured, visual book format.
- **Architecture coverage:** 10 specification files defining the full pipeline — content models, transformation rules, pedagogy rules, validation checklists, visual content model.
- **What I learned:** Vague transformation rules produce vague books. The spec-first approach isn't optional here — output quality is a direct function of how precisely the rules are written.

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
| `product-brief.md` | Prior product brief draft |
| `prd.md` | Product Requirements Document |
| `docs/product-brief.md` | Final product brief — source of truth |
| `Art Direction/` | Visual prototype outputs including Applyr deep-dive |

---

## Challenges & Decisions

### Rule precision as the primary quality lever
**Problem:** Asking an AI to "transform this content into a beginner-friendly book" produces inconsistent results. The output quality depends entirely on how precisely the transformation is defined.
**Decision:** Write 10 specification files before running the pipeline — content models, transformation rules, pedagogy rules, validation checklists. The AI executes the spec; it doesn't author it.
**Tradeoff:** Significant upfront spec work. The pipeline can't run until the rules are precise enough.
**Outcome:** Consistent, auditable output. Each output element traces back to a rule in the spec. When quality is wrong, the fix is a rule change, not a re-prompt.

### Validation before output
**Problem:** AI-generated content can pass a surface read and fail on quality — invented facts, skipped concepts, inconsistent terminology. Without a gate, bad output ships.
**Decision:** Validation checklists run automatically before any output is considered done. Quality gates are defined in the spec, not applied post-hoc.
**Tradeoff:** Slower pipeline. Validation adds time and can surface issues that require regenerating sections.
**Outcome:** Prototype output passed validation without manual review cycles. The checklist catches what a quick read misses.

---

## How This Was Built

I defined the architecture before writing any code. Product brief and PRD first, then content models and transformation rules to govern how the AI behaves. The spec-first approach isn't optional here: output quality depends on the rules being precise. Vague rules produce vague books.

The pipeline currently runs through AI-assisted execution of the guide files rather than automated software. Prototype outputs exist, including a vertical visual report on Applyr. Visual generation is functional. Art direction is still being refined.

---

## License

Private
