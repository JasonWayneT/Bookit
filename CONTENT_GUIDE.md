# Content Guide: Master Operating Instructions

## Document Control

**Version**: 1.0  
**Purpose**: Master operating guide for AI agents transforming source content into books  
**Scope**: Defines the entire content transformation pipeline  
**Status**: Active  

---

## 1. System Goal

Transform raw, uneven, expert-level content into clear, structured, beginner-accessible books formatted as vertical visual reports optimized for digital reading.

**Input**: Transcripts, notes, slides, expert discussions, technical documentation  
**Output**: Publication-ready books with pedagogical scaffolding, visual enhancements, and validated quality  

---

## 2. Content Pipeline

The transformation follows a strict six-stage pipeline. Each stage must complete before the next begins.

### Stage 1: Ingest
**Objective**: Accept and classify raw content  
**Actions**:
1. Receive source file (txt, md, pdf, docx, srt)
2. Extract text content
3. Classify content type using SOURCE_INPUT_TYPES.md
4. Create source_item metadata
5. Preserve original for attribution

**Output**: `source_item` with classified type and extracted text

### Stage 2: Structure
**Objective**: Build book hierarchy and identify gaps  
**Actions**:
1. Analyze content for major themes
2. Segment into chapters (one teaching objective each)
3. Create sections within chapters (one concept each)
4. Map prerequisite relationships
5. Detect content gaps (assumed knowledge not explained)
6. Generate hierarchical outline

**Output**: `book` metadata with parts/chapters/sections structure

### Stage 3: Expand
**Objective**: Transform raw content into beginner-accessible teaching prose  
**Actions**:
1. Detect all technical terms, jargon, acronyms
2. Define terms on first use (plain English + technical definition)
3. Add beginner scaffolding for complex concepts
4. Generate concrete examples after abstract explanations
5. Insert analogies where helpful
6. Add chapter recaps and transitions
7. Build glossary with metadata

**Output**: Expanded chapter text with inline definitions, glossary entries

### Stage 4: Illustrate
**Objective**: Add visual enhancements to support understanding  
**Actions**:
1. Identify visual candidates (processes, comparisons, systems, timelines)
2. Generate diagrams from text descriptions
3. Create charts from data mentions
4. Design callout boxes for key concepts
5. Add captions and alt text to all visuals
6. Store visual metadata

**Output**: Visual assets with metadata, updated chapter text with visual references

### Stage 5: Validate
**Objective**: Run automated quality checks  
**Actions**:
1. Execute pedagogical validation (terms defined, examples present, scaffolding added)
2. Check style consistency (tone, terminology, voice)
3. Verify content completeness (all required elements present)
4. Validate source attribution (content traces to sources)
5. Generate validation report with scores and failures

**Output**: `validation_report` with pass/fail results

### Stage 6: Publish
**Objective**: Render final vertical visual report  
**Actions**:
1. Apply design system (typography, colors, spacing)
2. Render responsive HTML optimized for scrolling
3. Embed visuals with proper placement
4. Generate table of contents and navigation
5. Export to PDF (optional)
6. Bundle downloadable assets

**Output**: HTML document, PDF file, public URL

---

## 3. Book Hierarchy

All books follow this structure:

```
Book
└── Part (optional, only if >50 pages)
    └── Chapter (one teaching objective)
        └── Section (one supporting concept)
            └── Subsection (focused explanation/example)
                └── Block (paragraph, list, visual, callout)
```

### Hierarchy Rules

**Book Level**:
- Must have: Title, introduction, promise to reader
- Optional: Dedication, preface
- Back matter: Glossary (required), references, appendices

**Part Level** (only for books >50 pages):
- Represents major thematic divisions
- Should have 3-7 chapters per part
- Must have: Part number, title, brief introduction

**Chapter Level**:
- **One teaching objective per chapter** (critical rule)
- Must have: Chapter number, title, opening hook, teaching objective statement
- Should have: Prerequisite concepts listed, key terms introduced, examples, chapter recap, bridge to next chapter
- Length: Typically 2,000-5,000 words

**Section Level**:
- **One supporting concept per section**
- Must have: Section heading, clear topic sentence
- Should explain: What, why, how, when to use this concept
- Length: Typically 300-800 words

**Subsection Level**:
- Focused on specific explanation, example, or application
- Must have: Subsection heading (optional if very short)
- Length: Typically 100-400 words

**Block Level**:
- Atomic content unit: paragraph, list, code block, visual, callout
- Should be: Self-contained, scannable, properly formatted

---

## 4. Reader-First Rule

**Core Principle**: The book must explain what the source assumes.

This means:
- If the source uses a term without definition, you must define it
- If the source jumps to advanced concepts, you must add scaffolding
- If the source references external knowledge, you must provide context
- If the source speaks expert-to-expert, you must translate to beginner-accessible language

**Test**: After every paragraph, ask: "Would a beginner understand this without prior knowledge?" If no, add explanation.

---

## 5. Expansion Principle

**Core Principle**: Do not merely preserve content; teach the subject.

This means:
- Raw transcripts are starting material, not final text
- Verbal filler ("um," "like," "you know") must be removed
- Tangents and repetition must be consolidated
- Implicit knowledge must be made explicit
- Abstract statements must be followed by concrete examples
- Complex ideas must be broken into digestible pieces

**Transformation Examples**:

**Before** (raw transcript):
"So, um, the thing about REST APIs is they're stateless, right? Which means, like, each request is independent, you know what I mean?"

**After** (expanded):
"REST APIs follow a stateless design. This means each API request is independent and contains all the information needed to process it. The server doesn't remember previous requests or maintain session state between calls. For example, when you refresh a web page, each refresh sends a complete HTTP request—the server doesn't 'remember' that you just loaded the same page a moment ago."

**Expansion Checklist**:
- ✅ Removed filler words
- ✅ Defined "stateless" in plain English
- ✅ Explained what it means in practice
- ✅ Provided concrete example
- ✅ Connected to familiar concept (web page refresh)

---

## 6. Contributor Workflow

### For Human Contributors

**When providing source content**:
1. Upload file via interface
2. Optionally specify: target audience level, desired book length, special instructions
3. Wait for AI processing (typically <1 hour for 20-page output)
4. Review output and validation report
5. Approve or request adjustments

**When reviewing output**:
1. Check validation report for flagged issues
2. Verify key claims are accurate
3. Test random sections for beginner comprehension
4. Provide feedback on quality
5. Approve for publication or request regeneration

### For AI Agents

**When processing source content**:
1. **READ ALL RELEVANT GUIDE FILES FIRST** (this file, BOOK_STYLE_MANIFEST, SOURCE_INPUT_TYPES, etc.)
2. Follow the six-stage pipeline in order
3. Do not skip stages or combine them
4. Record all transformation decisions in metadata
5. Generate validation report before final output
6. Log which guide file versions were used

**When uncertain**:
1. Flag the issue in metadata
2. Make best-effort transformation
3. Note uncertainty in validation report
4. Do NOT stop processing—continue and flag for human review

---

## 7. Definition of Ready (Input)

Source content is **ready for processing** when:

✅ File is uploaded and accessible  
✅ Text can be extracted successfully  
✅ Content type is classified (transcript, notes, article, etc.)  
✅ File size is within limits (<100,000 words)  
✅ No malware or corruption detected  

If content is **not ready**, system must:
- Reject upload with clear error message
- Provide guidance on fixing the issue
- Preserve partial extraction if possible

---

## 8. Definition of Done (Output)

Output is **done and ready for publication** when:

✅ **Structure**: All chapters have one clear teaching objective, sections have one concept each  
✅ **Vocabulary**: 90%+ of technical terms defined on first use, glossary is complete  
✅ **Examples**: Every abstract concept has a concrete example  
✅ **Scaffolding**: Advanced concepts have beginner scaffolding (prerequisites explained)  
✅ **Visuals**: Appropriate diagrams/charts/callouts are present (target: 1 visual per 500-1000 words)  
✅ **Style**: Tone matches BOOK_STYLE_MANIFEST, no forbidden patterns  
✅ **Completeness**: All chapters have required elements (hook, recap, examples)  
✅ **Validation**: Passes 80%+ of automated validation checks  
✅ **Attribution**: All content traces to source material  
✅ **Output**: Renders correctly as responsive HTML and exports to PDF  

If output **does not meet criteria**, system must:
- Generate validation report listing failures
- Re-process failed sections
- If still failing after 2 retries, flag for human review

---

## 9. Quality Gates

The system enforces three quality gates:

### Gate 1: After Structure (Stage 2)
**Check**: Does the outline make pedagogical sense?
- Each chapter has one teaching objective
- Prerequisites come before dependent concepts
- No circular dependencies
- Major gaps are identified

**Action**: If fails, regenerate structure with tighter constraints

### Gate 2: After Expansion (Stage 3)
**Check**: Is the content beginner-accessible?
- Technical terms are defined
- Examples follow explanations
- Scaffolding is present
- Tone matches style guide

**Action**: If fails, re-expand problem chapters

### Gate 3: After Validation (Stage 5)
**Check**: Does the complete book pass validation?
- 80%+ validation score
- No critical failures
- All required elements present

**Action**: If fails, regenerate failed components or flag for human review

---

## 10. Metadata Requirements

At every stage, the system must maintain:

**Source Tracking**:
- Original filename, upload date, file hash
- Content type classification
- Extraction method used

**Transformation Tracking**:
- Which guide file versions were used
- AI model and temperature used
- Processing timestamps
- Retry counts

**Quality Tracking**:
- Validation scores per category
- Flagged issues and resolutions
- Human review notes (if any)

**Output Tracking**:
- Generated file paths
- Public URLs
- Download counts
- User feedback

---

## 11. Error Handling

### Recoverable Errors
- Malformed input → Extract what's possible, flag rest
- Classification uncertainty → Use "notes" as default type
- Term definition failure → Skip definition, flag for human review
- Visual generation failure → Use text fallback, continue processing
- Validation failure → Regenerate failed component, retry once

### Unrecoverable Errors
- File corruption → Reject with error message
- Extraction complete failure → Reject with error message
- AI API unavailable → Retry with exponential backoff, then fail gracefully
- Out of memory → Chunk processing, reduce batch size

**Error Logging**:
All errors must be logged with:
- Error type and severity
- Stage where error occurred
- Book/chapter ID
- Timestamp
- Stack trace (if applicable)
- Resolution attempt (if any)

---

## 12. Performance Targets

**Processing Speed**:
- 20-page transcript → <1 hour end-to-end
- 100-page document → <5 hours end-to-end
- Visual generation → asynchronous, non-blocking

**Quality Targets**:
- Validation pass rate: 80%+
- Term definition accuracy: 90%+
- Human review time: <30 minutes per 20 pages
- Beginner comprehension improvement: 40%+ vs. raw content

**System Targets**:
- Uptime: 99%+
- API error rate: <1%
- Successful processing rate: 95%+

---

## 13. Continuous Improvement

The system learns and improves through:

**Feedback Loops**:
- Track validation failures by category
- Collect human review comments
- Monitor reader comprehension metrics
- Log common error patterns

**Guide File Updates**:
- Version all guide files in Git
- Test changes with A/B comparisons
- Roll back if quality degrades
- Document rationale for changes

**Model Updates**:
- Test new AI models periodically
- Benchmark against current quality
- Switch only if improvement >10%
- Maintain fallback to previous model

---

## 14. Success Metrics

**Quality Metrics**:
- Technical terms defined on first use: >90%
- Validation pass rate: >80%
- Example coverage: >90% of abstract concepts
- Visual coverage: 1 per 500-1000 words
- Forbidden pattern occurrences: <5 per book

**Efficiency Metrics**:
- Processing time (20 pages): <1 hour
- Human review time: <30 minutes per 20 pages
- Retry rate: <10%
- Error rate: <5%

**Outcome Metrics**:
- Reader retention (finish rate): >70%
- Comprehension improvement vs. raw content: >40%
- User satisfaction: >4/5 rating
- Content reuse rate: Track duplicate concept detection

---

## 15. Related Guide Files

This guide references and depends on:

- **BOOK_STYLE_MANIFEST.md**: Voice, tone, grammar rules
- **SOURCE_INPUT_TYPES.md**: Input classification and processing
- **BOOK_ARCHITECTURE.md**: Hierarchy and structure definitions
- **PEDAGOGY_RULES.md**: Teaching principles and scaffolding
- **VOCABULARY_AND_GLOSSARY_MODEL.md**: Term handling and glossary
- **VISUAL_CONTENT_MODEL.md**: Visual selection and generation
- **BOOK_TAXONOMY.md**: Classification systems
- **TRANSFORMATION_RULES.md**: Input-specific transformation logic
- **VALIDATION_CHECKLISTS.md**: Quality check definitions

AI agents must read relevant guide files before processing each stage.

---

## 16. Version History

- **v1.0** (2024-05-05): Initial content guide for AI-first transformation system
