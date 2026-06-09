---
title: Bookit Product Brief
status: draft
created: 2026-06-08
updated: 2026-06-08
---

# Bookit

> An AI content transformation pipeline that converts raw expert material — transcripts, notes, discussions — into structured, beginner-accessible digital books formatted as vertical visual reports.

---

## Executive Summary

Experts produce valuable content in forms that are inaccessible to the people who need it most: raw transcripts, fragmented notes, verbal explanations full of assumed knowledge. Bookit is a fully specified AI pipeline that ingests that raw content and outputs a publication-ready digital book — structured, visually rich, and designed for a beginner to actually understand.

The system is fully specified: 13 guide files defining the content model, pedagogy rules, transformation logic, validation checklists, visual model, and output format. Prototype outputs have been produced, including a deep-dive book on Applyr. Code implementation is planned.

---

## The Problem

Subject matter experts don't produce content in learnable formats. A consultant's meeting transcript, a podcaster's episode, a researcher's notes — all of it is valuable, all of it assumes prior knowledge, and none of it is structured for someone coming in fresh.

Turning raw expert content into something a beginner can absorb requires editorial work: restructuring, defining terms, adding context, creating visuals, sequencing concepts. That work is time-consuming, expensive, and inconsistently done.

The result: most expert knowledge stays locked in formats that only experts can use.

---

## The Solution

Bookit is a six-stage automated pipeline:

1. **Intake** — Classifies the source type (transcript, notes, PDF, slides), extracts text, preserves attribution metadata.
2. **Structure** — Builds the book hierarchy: chapters, sections, concept sequence. Detects gaps where explanation is missing.
3. **Expansion** — Adds beginner scaffolding: term definitions on first use, examples, explanations of assumed knowledge.
4. **Visual generation** — Produces diagrams, charts, and callout boxes from content — not decoration, but explanation rendered visually.
5. **Validation** — Runs automated checks against pedagogy rules: every term defined, visual-to-text ratio met, beginner comprehension standards passed.
6. **Output** — Renders a vertical visual report: full-width, scroll-optimized, designed for web and tablet reading.

Human provides raw content. The pipeline delivers a publication-ready book. Target: raw input to finished output in under one hour for a 20-page book, with under 30 minutes of human review required.

---

## What Makes This Different

Most content repurposing tools reformat. Bookit transforms. The distinction:

- **Pedagogy-first, not format-first.** The system is built around teaching principles — retrieval, scaffolding, concept sequencing — not layout templates.
- **Traceability.** Every output element maps back to source material. The pipeline doesn't invent content; it restructures and clarifies what's there.
- **Validation as a feature.** The output is checked against explicit rules before it's considered done. Quality isn't assumed; it's tested.

---

## Who This Serves

**Content creators with raw expertise:** Podcasters, educators, consultants who have valuable knowledge in unstructured form and no easy way to turn it into something publishable.

**Knowledge management teams:** Organizations with large libraries of internal documentation, transcripts, or training recordings that aren't accessible to the people who need them.

**Not the target:** Fiction writing, academic citation formatting, human-driven editorial workflows.

---

## Current State

The full system specification is built: 13 guide files covering content models, style manifest, taxonomy, pedagogy rules, transformation rules, validation checklists, visual content model, and chapter templates. Art direction prototypes have been produced, including a vertical visual report on Applyr.

Code implementation has not yet started. The pipeline currently runs through AI-assisted execution of the guide files rather than automated software. Visual output is still being refined — diagram and image generation is functional but the art direction approach is actively being worked out.

---

## Success Criteria

| Criterion | Target |
|---|---|
| Raw input to publication-ready output | < 1 hour for a 20-page book |
| Human review time required | < 30 minutes |
| Technical terms defined on first use | 90%+ |
| Automated validation pass rate | 80%+ |
| Reader comprehension vs. raw content | 40%+ improvement |

---

## Scope

**In scope:**
- Non-fiction content transformation (transcripts, notes, slides, PDFs)
- Beginner scaffolding and term definition
- Visual report output (HTML, exportable to PDF)
- Automated pedagogical validation

**Out of scope:**
- Fiction or creative writing
- Academic citation formats
- Multi-author collaboration
- Version control for ongoing content updates

---

## Reference Documents

| File | Purpose |
|---|---|
| `CONTENT_GUIDE.md` | Master content rules |
| `BOOK_STYLE_MANIFEST.md` | Visual and prose style |
| `PEDAGOGY_RULES.md` | Teaching principles baked into the pipeline |
| `TRANSFORMATION_RULES.md` | How raw content becomes structured output |
| `VALIDATION_CHECKLISTS.md` | Automated quality gates |
| `VISUAL_CONTENT_MODEL.md` | How visuals are generated from text |
| `Art Direction/` | Prototype outputs including Applyr deep-dive |
