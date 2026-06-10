# Book Taxonomy: Classification System Guide

## Document Control

**Version**: 1.0  
**Purpose**: Define classification systems for content and book units  
**Scope**: Content roles, reader levels, evidence types, editorial status  
**Status**: Active  

---

## 1. Content Role Taxonomy

### 1.1 Purpose

Every content block (paragraph, section, visual) serves a specific role. Classifying content by role helps ensure balanced coverage and proper sequencing.

### 1.2 Content Roles

**CONCEPT** - Introduces an abstract idea
- Example: "REST stands for Representational State Transfer"
- Placement: Early in section
- Must be followed by: DEFINITION or MECHANISM

**DEFINITION** - Explains what something is
- Example: "An API is a set of rules that lets applications communicate"
- Placement: At or before first use of term
- Must include: Plain-English explanation

**MECHANISM** - Explains how something works
- Example: "When a client sends a GET request, the server retrieves the resource and returns it in the response body"
- Placement: After CONCEPT/DEFINITION
- Must include: Step-by-step or cause-and-effect

**PROCESS** - Describes a sequence of steps
- Example: "To authenticate: 1) Send credentials, 2) Receive token, 3) Include token in subsequent requests"
- Placement: When teaching procedures
- Must include: Ordered steps

**EXAMPLE** - Concrete illustration of abstract concept
- Example: "fetch('https://api.example.com/users') retrieves all users"
- Placement: After CONCEPT/MECHANISM
- Must include: Real or realistic scenario

**ANALOGY** - Relates new concept to familiar one
- Example: "An API is like a restaurant menu—it lists what you can order"
- Placement: When introducing unfamiliar concepts
- Must include: Comparison to known concept

**EVIDENCE** - Supports a claim with data or citation
- Example: "A 2023 benchmark showed REST APIs handled 10,000 requests/second"
- Placement: When making factual claims
- Must include: Source or data

**CASE_STUDY** - Real-world application or story
- Example: "Twitter's API powers thousands of third-party apps"
- Placement: After theory, showing practical application
- Must include: Specific scenario

**MISCONCEPTION** - Addresses common misunderstanding
- Example: "Many think PUT and POST are the same, but PUT is idempotent while POST is not"
- Placement: After correct explanation
- Must include: Wrong belief + correct understanding

**WARNING** - Alerts to potential problems
- Example: "Never send passwords in URL query parameters"
- Placement: Before or alongside risky practices
- Must include: What to avoid and why

**EXERCISE** - Practice opportunity
- Example: "Try modifying the API call to request a different user"
- Placement: After explanation and examples
- Must include: Clear task

**VISUAL_CANDIDATE** - Text that should be visualized
- Example: "The request travels from client to server, the server processes it, then responds"
- Placement: Complex processes, relationships, comparisons
- Flags for: Diagram or chart generation

**BRIDGE** - Transitions between topics
- Example: "Now that we understand HTTP methods, let's explore status codes"
- Placement: Between sections or chapters
- Must include: Connection between topics

**GLOSSARY_ITEM** - Term requiring definition
- Example: "Idempotency (the property of producing the same result when repeated)"
- Placement: Inline on first use + in glossary
- Must include: Term + definition

---

## 2. Reader-Level Taxonomy

### 2.1 Purpose

Content can be targeted at different expertise levels. This taxonomy helps ensure beginners aren't overwhelmed and advanced readers aren't bored.

### 2.2 Reader Levels

**BEGINNER** - No prior knowledge assumed
- Characteristics:
  - All terms defined
  - Extensive scaffolding
  - Simple examples first
  - Step-by-step explanations
  - Frequent recaps
- Example: "An API is like a waiter in a restaurant..."

**PRACTITIONER** - Some experience, building skills
- Characteristics:
  - Basic terms can be used without definition
  - Moderate scaffolding
  - Real-world examples
  - Nuanced explanations
  - Best practices emphasized
- Example: "When designing RESTful endpoints, follow resource-oriented patterns..."

**ADVANCED** - Solid foundation, seeking mastery
- Characteristics:
  - Technical depth
  - Edge cases explored
  - Performance considerations
  - Architecture decisions
  - Trade-off discussions
- Example: "Horizontal scaling of stateless API servers requires consideration of distributed caching strategies..."

**EXPERT** - Deep knowledge, researching specifics
- Characteristics:
  - Formal specifications
  - Low-level implementation details
  - Research papers cited
  - Novel approaches
  - Theoretical foundations
- Example: "REST constraints as defined in Fielding's dissertation include..."

### 2.3 Level Transition Points

**Within a book**:
- Chapters 1-3: BEGINNER
- Chapters 4-7: PRACTITIONER
- Chapters 8-10: ADVANCED
- Appendices: EXPERT (optional)

**Within a chapter**:
- Opening: BEGINNER (establish foundation)
- Middle: PRACTITIONER (build on basics)
- Advanced section (optional): ADVANCED (for curious readers)

**Flag content with level**:
```yaml
section:
  title: "Advanced: Optimizing API Performance"
  reader_level: ADVANCED
  optional: true
  prerequisite_sections: ["basics", "intermediate"]
```

---

## 3. Evidence Taxonomy

### 3.1 Purpose

When making claims, classify the type of evidence to ensure appropriate confidence and attribution.

### 3.2 Evidence Types

**PRIMARY_SOURCE** - Original research or specification
- Example: "RFC 7231 defines HTTP methods"
- Confidence: High
- Citation: Required
- Use when: Citing standards, original papers

**SECONDARY_SOURCE** - Analysis or commentary
- Example: "According to InfoQ's 2023 API report..."
- Confidence: Medium-High
- Citation: Required
- Use when: Citing industry reports, expert analysis

**ANECDOTE** - Personal experience or observation
- Example: "In my experience building APIs..."
- Confidence: Low (individual case)
- Citation: State it's anecdotal
- Use when: Illustrating concepts, not proving facts

**DATA** - Quantitative measurements
- Example: "Benchmark showed 150ms average response time"
- Confidence: High (if methodology sound)
- Citation: Required (source + methodology)
- Use when: Performance claims, comparisons

**AUTHOR_INTERPRETATION** - Book author's analysis
- Example: "This suggests that stateless design improves scalability"
- Confidence: Medium
- Citation: Mark as interpretation
- Use when: Drawing conclusions, offering opinions

**HYPOTHESIS** - Unproven claim or prediction
- Example: "This approach may reduce latency in high-concurrency scenarios"
- Confidence: Low
- Citation: Mark as hypothesis
- Use when: Speculating, proposing ideas

### 3.3 Evidence Strength Indicators

**Strong evidence**:
- Primary sources (specifications, standards)
- Peer-reviewed research
- Large-scale benchmarks with methodology
- Multiple corroborating sources

**Weak evidence**:
- Single anecdotes
- Unsourced claims
- Hypothetical examples
- "Common knowledge" without verification

### 3.4 Evidence Metadata

```yaml
evidence:
  type: enum [primary_source, secondary_source, anecdote, data, author_interpretation, hypothesis]
  claim: text
  source: string (citation or "author analysis")
  confidence: enum [high, medium, low]
  verification_status: enum [verified, unverified, disputed]
```

---

## 4. Editorial Status Taxonomy

### 4.1 Purpose

Track the transformation state of content through the pipeline.

### 4.2 Status States

**RAW** - Uploaded, not yet processed
- Content: Original source file
- Next step: Classification
- Can publish: No

**CLASSIFIED** - Type identified, metadata created
- Content: Source + type classification
- Next step: Structure generation
- Can publish: No

**DRAFTED** - Initial structure created
- Content: Outline with placeholders
- Next step: Expansion
- Can publish: No

**EXPANDED** - Content transformed to teaching prose
- Content: Full text with definitions, examples
- Next step: Visual generation
- Can publish: Maybe (basic version)

**ILLUSTRATED** - Visuals generated and integrated
- Content: Text + diagrams/charts/callouts
- Next step: Validation
- Can publish: Maybe (if validation passes)

**REVIEWED** - Validation run, issues flagged
- Content: Complete book + validation report
- Next step: Fix issues or approve
- Can publish: If score >80%

**VALIDATED** - Passes quality checks
- Content: Complete, validated book
- Next step: Final output generation
- Can publish: Yes

**READY** - Final output generated
- Content: HTML/PDF published
- Next step: None
- Can publish: Yes (already published)

### 4.3 Status Transitions

```
RAW → CLASSIFIED → DRAFTED → EXPANDED → ILLUSTRATED → REVIEWED → VALIDATED → READY
         ↓              ↓          ↓            ↓            ↓           ↓
       FAILED       FAILED     FAILED       FAILED      FAILED     NEEDS_REVISION
```

**FAILED** - Error during processing
- Requires: Manual intervention or retry
- Next step: Fix issue and restart from failure point

**NEEDS_REVISION** - Validation failed, requires re-processing
- Requires: Update content and re-run validation
- Next step: Return to appropriate stage (EXPANDED or ILLUSTRATED)

---

## 5. Structural Taxonomy

### 5.1 Hierarchy Levels

**BOOK** - Top-level container
- Contains: Parts (optional), Chapters, Front matter, Back matter
- Teaching goal: Complete subject coverage
- Reader outcome: Competency in topic

**PART** (Optional, for books >50 pages)
- Contains: Chapters
- Teaching goal: Major thematic division
- Reader outcome: Understanding of one major aspect

**CHAPTER** - Primary teaching unit
- Contains: Sections, Subsections
- Teaching goal: One primary objective
- Reader outcome: Ability to explain/apply one concept

**SECTION** - Supporting teaching unit
- Contains: Subsections, Blocks
- Teaching goal: One supporting concept
- Reader outcome: Understanding of one sub-concept

**SUBSECTION** - Focused explanation
- Contains: Blocks
- Teaching goal: Specific detail or example
- Reader outcome: Detailed understanding of facet

**BLOCK** - Atomic content unit
- Contains: Paragraph, List, Code, Visual, Callout
- Teaching goal: Single point or element
- Reader outcome: Comprehension of specific information

---

## 6. Metadata Classification Examples

### Example 1: Paragraph Classification

```yaml
block:
  type: paragraph
  content_role: DEFINITION
  reader_level: BEGINNER
  contains_terms: ["API"]
  first_use_terms: ["API"]
```

### Example 2: Section Classification

```yaml
section:
  title: "Understanding HTTP Methods"
  content_roles: [CONCEPT, DEFINITION, MECHANISM, EXAMPLE]
  reader_level: BEGINNER
  evidence_used: [SECONDARY_SOURCE]
  terms_introduced: ["HTTP method", "GET", "POST", "PUT", "DELETE"]
  visual_count: 1
```

### Example 3: Chapter Classification

```yaml
chapter:
  number: 2
  title: "HTTP Methods and Resources"
  teaching_objective: "Understand how HTTP methods map to CRUD operations"
  reader_level: BEGINNER
  content_role_distribution:
    CONCEPT: 5
    DEFINITION: 8
    MECHANISM: 6
    EXAMPLE: 10
    ANALOGY: 2
    MISCONCEPTION: 1
  editorial_status: VALIDATED
  validation_score: 0.87
```

---

## 7. Balanced Content Mix

### 7.1 Healthy Chapter Ratios

**For beginner content**:
- DEFINITION: 15-25% (many terms to define)
- EXAMPLE: 30-40% (concrete illustrations critical)
- MECHANISM: 20-30% (how things work)
- CONCEPT: 10-15% (introduce ideas)
- ANALOGY: 5-10% (aid understanding)
- MISCONCEPTION: 5-10% (address confusion)

**For advanced content**:
- MECHANISM: 30-40% (detailed how-it-works)
- EVIDENCE: 20-30% (data and citations)
- CASE_STUDY: 15-25% (real applications)
- EXAMPLE: 15-20% (still needed but fewer)
- DEFINITION: 5-10% (fewer new terms)

### 7.2 Warning Signs

**Too many DEFINITIONS, too few EXAMPLES**:
- Reads like a dictionary, not a teaching book
- Fix: Add concrete examples after each definition

**Too many CONCEPTS, not enough MECHANISM**:
- Introduces ideas without explaining how they work
- Fix: Add how-it-works explanations

**EXAMPLES without preceding DEFINITIONS**:
- Confusing if reader doesn't know what's being illustrated
- Fix: Define terms before showing examples

**No BRIDGES**:
- Choppy, disconnected sections
- Fix: Add transitions between topics

---

## 8. Version History

- **v1.0** (2024-05-05): Initial book taxonomy guide
