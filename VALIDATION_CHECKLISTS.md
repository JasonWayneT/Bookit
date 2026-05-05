# Validation Checklists: Quality Gate Definitions

## Document Control

**Version**: 1.0  
**Purpose**: Define all quality checks and pass/fail criteria  
**Scope**: Source intake, architecture, chapter, vocabulary, visual, style, pedagogy, final publish  
**Status**: Active  

---

## How to Use These Checklists

**For AI agents**: Run these checklists automatically. Each checklist returns:
- Pass/fail per check
- Overall score (0.0-1.0)
- List of failed checks with details
- Recommendations for fixes

**Pass threshold**: 80% (0.8) for most checklists  
**Critical failures**: Any check marked CRITICAL must pass  

**Workflow**:
1. Complete relevant stage (intake, expansion, etc.)
2. Run corresponding checklist
3. If score < 0.8 or critical failure: Fix issues and rerun
4. If score >= 0.8 and no critical failures: Proceed to next stage

---

## 1. Source Intake Checklist

**Run after**: File upload and classification  
**Pass threshold**: 100% (all checks must pass)  

### File Integrity

- [ ] **CRITICAL**: File uploaded successfully
- [ ] **CRITICAL**: File is readable (not corrupted)
- [ ] **CRITICAL**: File size within limits (<100MB)
- [ ] File encoding detected (UTF-8, ASCII, etc.)
- [ ] No malware detected

**Fail actions**: Reject file with error message

### Content Extraction

- [ ] **CRITICAL**: Text extracted successfully
- [ ] Word count > 0
- [ ] If PDF: Text extracted (not image-only scan)
- [ ] If DOCX: Structure preserved
- [ ] If SRT: Timestamps and speakers parsed

**Fail actions**: Flag for manual review or alternative extraction method

### Classification

- [ ] Content type classified (transcript, notes, article, etc.)
- [ ] Classification confidence >= 0.7
- [ ] If confidence < 0.7: Flagged for manual review
- [ ] Topic keywords extracted

**Fail actions**: Default to "notes" type if classification fails

### Metadata Creation

- [ ] Source ID generated
- [ ] File hash computed
- [ ] Upload date recorded
- [ ] All required metadata fields populated

**Overall score calculation**:
```
score = (checks_passed / total_checks)
pass = score == 1.0 AND no_critical_failures
```

---

## 2. Book Architecture Checklist

**Run after**: Structure generation  
**Pass threshold**: 80%  

### Book Promise

- [ ] **CRITICAL**: Teaching goal defined
- [ ] Audience level specified (beginner/practitioner/advanced)
- [ ] Transformation objective stated
- [ ] Scope included and excluded defined
- [ ] Payoff articulated

### Hierarchy

- [ ] **CRITICAL**: Chapter count >= 3
- [ ] Each chapter has ONE teaching objective
- [ ] Chapter objectives are specific and testable
- [ ] Parts used appropriately (if >50 pages) or not used (if <50 pages)
- [ ] Section count per chapter: 3-6
- [ ] Subsection usage is appropriate (not excessive)

### Prerequisites

- [ ] **CRITICAL**: No circular dependencies (Chapter A requires B which requires A)
- [ ] Prerequisites come before dependent chapters
- [ ] Prerequisite concepts identified for each chapter
- [ ] Complexity gradient: simple → complex

### Sequencing

- [ ] Logical progression pattern chosen
- [ ] Chapters flow naturally
- [ ] Each chapter bridges to next
- [ ] Motivation before mechanism

### Gaps

- [ ] Content gaps identified (missing prerequisite explanations)
- [ ] No chapters orphaned (all connect to book promise)
- [ ] Front matter planned (intro, ToC)
- [ ] Back matter planned (glossary)

**Overall score calculation**:
```
score = (checks_passed / total_checks)
pass = score >= 0.8 AND no_critical_failures
```

---

## 3. Chapter Readiness Checklist

**Run after**: Chapter expansion  
**Pass threshold**: 80%  

### Structure

- [ ] **CRITICAL**: Chapter number and title present
- [ ] **CRITICAL**: Teaching objective stated clearly
- [ ] Opening hook engages reader
- [ ] Prerequisites listed
- [ ] Chapter structure previewed
- [ ] Core content sections: 3-6
- [ ] Chapter recap summarizes key points
- [ ] Bridge to next chapter present

### Pedagogy

- [ ] **CRITICAL**: One primary teaching objective (not multiple)
- [ ] **CRITICAL**: All technical terms defined before or at first use
- [ ] Every abstract concept has concrete example
- [ ] Examples follow explanations (not before)
- [ ] Scaffolding provided for complex concepts
- [ ] Prerequisites explained or referenced
- [ ] Hidden assumptions detected and addressed

### Length

- [ ] Word count: 2,000-5,000 words
- [ ] If <1,500 words: Flagged for expansion
- [ ] If >7,000 words: Flagged for splitting

### Examples

- [ ] **CRITICAL**: At least 3 examples included
- [ ] Examples are realistic (not foo/bar/baz)
- [ ] Examples are complete (not fragments)
- [ ] Examples vary in complexity
- [ ] Code examples have comments/explanations

### Terms

- [ ] All terms used are defined somewhere
- [ ] Terms in "Key Terms" section are actually used in chapter
- [ ] No jargon stacking (multiple undefined terms in one sentence)
- [ ] Acronyms spelled out on first use

### Visuals

- [ ] At least 1 visual planned or present
- [ ] Visual-to-text ratio appropriate (target: 1 per 500-1000 words)
- [ ] Each visual has purpose (not decoration)

### Completeness

- [ ] Opening → objective → sections → examples → recap → bridge (all present)
- [ ] Common misconceptions addressed (if applicable)
- [ ] Practice opportunities included (if helpful)
- [ ] All promised topics in preview are covered

**Overall score calculation**:
```
score = (checks_passed / total_checks)
pass = score >= 0.8 AND no_critical_failures
```

---

## 4. Vocabulary/Glossary Checklist

**Run after**: Expansion stage  
**Pass threshold**: 90%  

### Term Detection

- [ ] **CRITICAL**: All technical terms identified
- [ ] All acronyms identified
- [ ] Domain-specific jargon flagged
- [ ] Overloaded words (common words with technical meanings) identified

### First-Use Compliance

- [ ] **CRITICAL**: 90%+ of technical terms defined on first use
- [ ] Definitions are plain-English (not circular jargon)
- [ ] Acronyms spelled out on first use
- [ ] Context provided for overloaded terms

### Definition Quality

- [ ] Plain-English definitions provided
- [ ] Definitions are 1-2 sentences (not overly long)
- [ ] Technical definitions added where helpful
- [ ] Examples included for complex terms

### Glossary Completeness

- [ ] All defined terms in glossary database
- [ ] Metadata complete (chapter, definition, etc.)
- [ ] Related terms identified
- [ ] Aliases/synonyms tracked

### Consistency

- [ ] **CRITICAL**: Same term used consistently (no unexplained synonym switching)
- [ ] Canonical terms match glossary
- [ ] Terms from earlier chapters reused consistently

**Overall score calculation**:
```
score = (checks_passed / total_checks)
pass = score >= 0.9 AND no_critical_failures
```

---

## 5. Visual Content Checklist

**Run after**: Visual generation  
**Pass threshold**: 80%  

### Visual Selection

- [ ] **CRITICAL**: Visuals serve purpose (not decoration)
- [ ] Visual-to-text ratio: ~1 per 500-1000 words
- [ ] Visuals placed near relevant text
- [ ] Each visual reduces cognitive load or reveals structure

### Visual Quality

- [ ] Resolution adequate (text readable)
- [ ] Charts: axes labeled with units
- [ ] Diagrams: all elements labeled
- [ ] Tables: headers clear
- [ ] Colors meet accessibility standards (WCAG AA contrast)

### Metadata

- [ ] **CRITICAL**: Each visual has caption
- [ ] **CRITICAL**: Each visual has alt text
- [ ] Figure numbers assigned (e.g., "Figure 3.2")
- [ ] Visuals referenced in nearby prose

### Integration

- [ ] Text introduces visual
- [ ] Visual appears at appropriate point
- [ ] Text explains key insights from visual
- [ ] Connection between text and visual is clear

### Accuracy

- [ ] **CRITICAL**: Data is real (or marked as illustrative)
- [ ] Labels are correct
- [ ] Relationships accurately depicted
- [ ] Citations provided for real data

### Style

- [ ] Visual style matches book design system
- [ ] Consistent with other visuals
- [ ] Fonts/colors/spacing appropriate
- [ ] Accessible (color-blind safe if possible)

**Overall score calculation**:
```
score = (checks_passed / total_checks)
pass = score >= 0.8 AND no_critical_failures
```

---

## 6. Tone and Style Checklist

**Run after**: Expansion/revision  
**Pass threshold**: 85%  

### Voice

- [ ] **CRITICAL**: No forbidden phrases (0 instances of "obviously," "simply," "just," "clearly," "of course")
- [ ] Patient, not rushed
- [ ] Encouraging, not dismissive
- [ ] Confident but humble (acknowledges complexity)
- [ ] Precise but accessible

### Sentence Structure

- [ ] **CRITICAL**: No jargon stacking
- [ ] Average sentence length: 10-25 words
- [ ] Active voice preferred (passive only when appropriate)
- [ ] Positive phrasing where possible

### Paragraph Structure

- [ ] One concept per paragraph
- [ ] Paragraph length: 3-6 sentences (mostly)
- [ ] No dense walls of text (>8 sentences)

### Consistency

- [ ] Same terms used throughout
- [ ] No unexplained synonym alternation
- [ ] Tone consistent across chapters

### Beginner-Friendliness

- [ ] Assumes intelligence, not prior vocabulary
- [ ] Technical terms explained before use
- [ ] No condescending language
- [ ] Examples are concrete and relatable

**Overall score calculation**:
```
score = (checks_passed / total_checks)
pass = score >= 0.85 AND no_critical_failures
```

---

## 7. Pedagogical Completeness Checklist

**Run after**: Expansion stage  
**Pass threshold**: 80%  

### Scaffolding

- [ ] **CRITICAL**: Prerequisites stated for chapters
- [ ] Concepts build from simple to complex
- [ ] No unexplained prerequisite assumptions
- [ ] Scaffolding provided for advanced concepts

### Concept Ladder

- [ ] Familiar idea → new term → mechanism → example → application pattern followed
- [ ] One major idea per chapter
- [ ] One concept per section
- [ ] Progressive disclosure used

### Examples

- [ ] **CRITICAL**: Every abstract concept has concrete example
- [ ] Examples are realistic
- [ ] Examples are complete
- [ ] Examples vary in complexity (basic → advanced)

### Misconceptions

- [ ] Common misconceptions addressed (if applicable)
- [ ] Incorrect beliefs stated clearly
- [ ] Correct understanding provided
- [ ] Examples demonstrate correct approach

### Cognitive Load

- [ ] Maximum 3-5 new terms per section
- [ ] Visuals used to reduce text density
- [ ] White space adequate
- [ ] Complex ideas broken into digestible pieces

### Recaps and Reinforcement

- [ ] Chapter recaps present
- [ ] Key points summarized (3-7 items)
- [ ] Concepts from earlier chapters referenced
- [ ] Spaced repetition used

**Overall score calculation**:
```
score = (checks_passed / total_checks)
pass = score >= 0.8 AND no_critical_failures
```

---

## 8. Citation/Evidence Checklist

**Run after**: Validation stage  
**Pass threshold**: 85%  

### Attribution

- [ ] **CRITICAL**: All factual claims trace to sources
- [ ] Citations formatted correctly
- [ ] Sources are credible
- [ ] No unsupported factual claims

### Evidence Types

- [ ] Evidence type identified (primary, secondary, anecdote, data)
- [ ] Appropriate evidence for claim strength
- [ ] Anecdotes marked as such (not presented as facts)
- [ ] Hypotheses marked as such

### Source Quality

- [ ] Primary sources used for specifications
- [ ] Peer-reviewed sources for research claims
- [ ] Benchmarks include methodology
- [ ] No broken links

### Verification

- [ ] Factual claims verified
- [ ] Conflicting sources acknowledged
- [ ] Author interpretations marked as such
- [ ] Data sources cited

**Overall score calculation**:
```
score = (checks_passed / total_checks)
pass = score >= 0.85 AND no_critical_failures
```

---

## 9. Final Publish-Readiness Checklist

**Run before**: Final output generation  
**Pass threshold**: 90%  

### Content Completeness

- [ ] **CRITICAL**: All chapters present and complete
- [ ] Front matter complete (title, intro, ToC)
- [ ] Back matter complete (glossary)
- [ ] All visuals present
- [ ] All code examples tested (if applicable)

### Quality Gates

- [ ] **CRITICAL**: Architecture checklist passed
- [ ] **CRITICAL**: 80%+ of chapters passed chapter checklist
- [ ] Vocabulary checklist passed
- [ ] Visual checklist passed
- [ ] Style checklist passed
- [ ] Pedagogy checklist passed

### Technical Validation

- [ ] Word count: 15,000-100,000 words
- [ ] Chapter count: 3-20 chapters
- [ ] Glossary: 20+ terms
- [ ] Visual count: 10+ visuals
- [ ] Code examples: syntax valid (if applicable)

### Accessibility

- [ ] All visuals have alt text
- [ ] Color contrast meets WCAG AA
- [ ] Heading hierarchy proper (H1 → H2 → H3)
- [ ] Links descriptive (not "click here")

### Formatting

- [ ] Consistent heading levels
- [ ] Code blocks formatted properly
- [ ] Lists formatted consistently
- [ ] Tables render correctly

### Metadata

- [ ] Book title, subtitle present
- [ ] Author/source attribution
- [ ] Version number
- [ ] Publication date
- [ ] Table of contents auto-generated

**Overall score calculation**:
```
score = (checks_passed / total_checks)
pass = score >= 0.9 AND no_critical_failures
```

---

## Checklist Scoring System

### Score Calculation

```python
def calculate_checklist_score(checks):
    total = len(checks)
    passed = sum(1 for check in checks if check.passed)
    critical_failures = sum(1 for check in checks if check.critical and not check.passed)
    
    score = passed / total
    pass_checklist = (score >= threshold) and (critical_failures == 0)
    
    return {
        'score': score,
        'passed': pass_checklist,
        'checks_passed': passed,
        'checks_failed': total - passed,
        'critical_failures': critical_failures
    }
```

### Severity Levels

**CRITICAL**: Must pass (blocks proceeding if failed)
- Examples: File readable, teaching objective present, terms defined

**HIGH**: Should pass (warns but doesn't block)
- Examples: Examples included, scaffolding provided

**MEDIUM**: Nice to have
- Examples: Practice exercises, advanced sections

### Fail Actions

**If checklist fails**:
1. Log all failed checks with details
2. Generate recommendations for fixes
3. If critical failure: Block progression, flag for manual review
4. If non-critical: Allow progression but note issues
5. Retry count: Maximum 3 attempts per checklist

**Retry logic**:
```python
def validate_with_retry(content, checklist, max_retries=3):
    for attempt in range(max_retries):
        result = run_checklist(content, checklist)
        if result['passed']:
            return result
        
        # Auto-fix if possible
        fixes_applied = apply_auto_fixes(content, result['failed_checks'])
        if not fixes_applied and attempt == max_retries - 1:
            flag_for_manual_review(content, result)
            break
    
    return result
```

---

## Checklist Weights

When computing overall book score, weight checklists:

```yaml
checklist_weights:
  architecture: 0.20  # Structure is foundational
  chapter_readiness: 0.25  # Content quality is critical
  vocabulary: 0.15  # Term handling matters
  visual: 0.10  # Visuals enhance but aren't core
  style: 0.15  # Voice and tone important
  pedagogy: 0.15  # Teaching effectiveness critical
```

Overall book score:
```
overall_score = sum(checklist_score * weight for all checklists)
pass = overall_score >= 0.8 AND all_critical_checklists_passed
```

---

## Version History

- **v1.0** (2024-05-05): Initial validation checklists guide
