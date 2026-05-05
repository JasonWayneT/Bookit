# Continuity Bible Template: Cross-Book Consistency Guide

## Document Control

**Version**: 1.0  
**Purpose**: Maintain consistency across books or series  
**Scope**: Terminology, concepts, examples, visuals, decisions  
**Status**: Active  

---

## Purpose of the Continuity Bible

The Continuity Bible ensures consistency when:
- Generating multiple books from overlapping content
- Creating book series on related topics
- Expanding or updating existing books
- Maintaining terminology across organization

**Key principle**: Readers should encounter the same definitions, examples, and explanations for concepts that appear in multiple books.

---

## 1. Canonical Terminology

### 1.1 Master Term List

**Purpose**: Define the ONE correct term to use across all books

**Format**:

```yaml
canonical_terms:
  - term: "REST API"
    status: canonical
    aliases: ["RESTful API", "RESTful web service", "REST endpoint"]
    definition: "An API that follows REST architectural principles"
    first_introduced: "book_001"
    used_in_books: ["book_001", "book_002", "book_003"]
    notes: "Use 'REST API' consistently. Mention aliases once if helpful."
  
  - term: "HTTP method"
    status: canonical
    aliases: ["HTTP verb", "request method"]
    definition: "A token in an HTTP request specifying the action to perform"
    first_introduced: "book_001"
    used_in_books: ["book_001", "book_002"]
    notes: "Prefer 'HTTP method' over 'HTTP verb'"
  
  - term: "endpoint"
    status: canonical
    aliases: ["API endpoint", "URL endpoint"]
    definition: "A specific URL where an API can be accessed"
    first_introduced: "book_001"
    used_in_books: ["book_001", "book_002", "book_003"]
    notes: "Can use 'endpoint' alone after first introduction"
```

### 1.2 Deprecated Terms

**Purpose**: Track terms no longer used

```yaml
deprecated_terms:
  - old_term: "RESTful service"
    new_term: "REST API"
    deprecated_date: "2024-05-01"
    reason: "Simplifying terminology"
    books_affected: ["book_002", "book_003"]
    migration_note: "Replace in future editions"
```

### 1.3 Term Evolution

**Purpose**: Track how definitions evolve across books

```yaml
term_evolution:
  - term: "API"
    book_001_definition: "A set of rules that lets software communicate"
    book_002_definition: "A specification defining operations, parameters, and responses for software interaction"
    evolution_note: "Book 1 is beginner-friendly, Book 2 is more technical. Both correct, different depth."
```

---

## 2. Recurring Concepts

### 2.1 Core Concepts List

**Purpose**: Ensure consistent explanation of fundamental ideas

```yaml
recurring_concepts:
  - concept: "Statelessness in REST"
    canonical_explanation: "Each request contains all information needed to process it. Server doesn't remember previous requests."
    analogy: "Restaurant waiter analogy (each order is complete)"
    first_explained: "book_001_chapter_02"
    subsequent_appearances:
      - book_id: "book_002"
        context: "Advanced statelessness patterns"
        treatment: "Brief refresher + advanced details"
      - book_id: "book_003"
        context: "Stateless authentication"
        treatment: "Reference to Book 1, apply to auth"
    
  - concept: "Idempotency"
    canonical_explanation: "Repeating an operation produces the same result"
    analogy: "Light switch analogy (flipping 'on' multiple times still just 'on')"
    first_explained: "book_001_chapter_02"
    subsequent_appearances:
      - book_id: "book_002"
        context: "Idempotent retry strategies"
        treatment: "Deep dive into implementation"
```

### 2.2 Concept Relationships

**Purpose**: Map dependencies between concepts

```yaml
concept_dependencies:
  - concept: "OAuth 2.0"
    prerequisite_concepts:
      - "HTTP methods"
      - "Tokens"
      - "Statelessness"
    builds_to:
      - "OAuth 2.0 flows"
      - "Token refresh"
    introduced_in: "book_001_chapter_07"
```

---

## 3. Standard Definitions

### 3.1 Beginner Definitions

**Purpose**: Consistent beginner-friendly explanations

```yaml
beginner_definitions:
  - term: "API"
    definition: "A set of rules that lets different software applications talk to each other"
    example: "Weather app uses API to get forecast from weather service"
    used_in: ["book_001", "book_002_intro"]
  
  - term: "HTTP"
    definition: "The protocol (set of rules) that web browsers and servers use to communicate"
    example: "When you type a URL, your browser uses HTTP to request the page"
    used_in: ["book_001", "book_002"]
```

### 3.2 Technical Definitions

**Purpose**: Precise definitions for advanced content

```yaml
technical_definitions:
  - term: "REST"
    definition: "An architectural style for distributed hypermedia systems, defined by Roy Fielding in 2000, characterized by statelessness, client-server architecture, cacheability, layered system, uniform interface, and optional code-on-demand"
    specification: "Fielding dissertation chapter 5"
    used_in: ["book_002", "book_003"]
```

---

## 4. Standard Examples

### 4.1 Example Library

**Purpose**: Reuse good examples across books

```yaml
example_library:
  - id: "example_001"
    title: "User CRUD operations"
    concept: "HTTP methods mapping to CRUD"
    code: |
      GET /users/123    # Read
      POST /users       # Create
      PUT /users/123    # Update
      DELETE /users/123 # Delete
    explanation: "Demonstrates how HTTP methods map to database operations"
    used_in: ["book_001_ch02", "book_002_ch01"]
    variations:
      - variation: "With authentication headers"
        book: "book_001_ch07"
      - variation: "With query parameters"
        book: "book_002_ch03"
  
  - id: "example_002"
    title: "Weather API request"
    concept: "GET requests with parameters"
    code: |
      GET https://api.weather.com/v1/current?city=Seattle
    explanation: "Shows query parameter usage in URLs"
    used_in: ["book_001_ch01", "book_003_ch02"]
```

### 4.2 Anti-Examples

**Purpose**: Consistent "what not to do" examples

```yaml
anti_examples:
  - id: "anti_001"
    title: "Passwords in URL"
    concept: "Security mistake"
    bad_code: "GET /login?username=alice&password=secret123"
    explanation: "Never send passwords in URLs—they appear in logs"
    correct_approach: "Use POST with HTTPS and body payload"
    used_in: ["book_001_ch07", "book_002_ch05"]
```

---

## 5. Standard Analogies

### 5.1 Analogy Registry

**Purpose**: Reuse effective analogies

```yaml
analogies:
  - id: "analogy_001"
    concept: "API"
    analogy: "Restaurant menu"
    explanation: "API is like a menu—lists what you can order (operations), what info to provide (parameters), what you'll get (responses)"
    limitations: "Unlike menus, APIs can handle millions of requests simultaneously"
    used_in: ["book_001_ch01", "book_002_intro"]
    effectiveness: "high"
  
  - id: "analogy_002"
    concept: "Statelessness"
    analogy: "Restaurant waiter"
    explanation: "Waiter doesn't remember your previous orders. Each order must be complete (appetizer, main, drink)"
    limitations: "Real waiters do remember regulars; APIs truly don't maintain state"
    used_in: ["book_001_ch02"]
    effectiveness: "medium"
```

---

## 6. Visual Conventions

### 6.1 Diagram Standards

**Purpose**: Consistent visual language

```yaml
diagram_standards:
  colors:
    client: "#3B82F6"  # Blue
    server: "#10B981"  # Green
    data: "#F59E0B"    # Orange
    error: "#EF4444"   # Red
  
  shapes:
    component: "rectangle"
    decision: "diamond"
    process: "rounded_rectangle"
    data_store: "cylinder"
  
  arrows:
    request: "solid_line"
    response: "dashed_line"
    data_flow: "thick_arrow"
  
  labels:
    font: "sans-serif"
    size: "14px"
    color: "#1F2937"
```

### 6.2 Chart Standards

**Purpose**: Consistent chart styling

```yaml
chart_standards:
  bar_charts:
    colors: ["#3B82F6", "#10B981", "#F59E0B", "#EF4444", "#8B5CF6"]
    bar_width: "0.8"
    show_values: true
  
  line_charts:
    line_width: "2px"
    point_size: "6px"
    grid: true
  
  general:
    title_font_size: "16px"
    axis_label_font_size: "12px"
    legend_position: "top_right"
```

---

## 7. Reader Assumptions

### 7.1 Prerequisite Knowledge by Book

**Purpose**: Track what readers should know for each book

```yaml
reader_assumptions:
  - book_id: "book_001"
    book_title: "REST APIs for Beginners"
    assumes_knowledge:
      - "Basic computer literacy"
      - "Web browsing experience"
      - "None: programming, APIs, HTTP"
    target_audience: "Complete beginners"
  
  - book_id: "book_002"
    book_title: "Advanced REST API Design"
    assumes_knowledge:
      - "Completed Book 001 or equivalent"
      - "HTTP methods, status codes"
      - "JSON format"
      - "Basic programming (any language)"
    target_audience: "Practitioners"
```

### 7.2 Knowledge Progression

**Purpose**: Map knowledge building across series

```yaml
knowledge_progression:
  foundation: "book_001"
  builds_to:
    - book_id: "book_002"
      adds: ["Advanced patterns", "Performance optimization", "Security hardening"]
    - book_id: "book_003"
      adds: ["Microservices", "GraphQL comparison", "API governance"]
```

---

## 8. Open Questions

### 8.1 Unresolved Issues

**Purpose**: Track questions needing resolution

```yaml
open_questions:
  - question: "Should we standardize on 'endpoint' or 'API endpoint'?"
    date_raised: "2024-05-01"
    status: "under_review"
    implications: "Affects book_002 and book_003"
    proposed_resolution: "Use 'endpoint' after first introduction"
  
  - question: "Best analogy for OAuth flows?"
    date_raised: "2024-05-03"
    status: "researching"
    current_analogy: "Hotel key card system"
    concerns: "May be too complex"
```

---

## 9. Decision Log

### 9.1 Editorial Decisions

**Purpose**: Record why choices were made

```yaml
decisions:
  - decision_id: "dec_001"
    date: "2024-05-01"
    decision: "Use 'REST API' as canonical term"
    rationale: "Most commonly used, clearest for beginners"
    alternatives_considered: ["RESTful API", "REST service"]
    impact: "All books must use 'REST API' consistently"
    status: "approved"
  
  - decision_id: "dec_002"
    date: "2024-05-02"
    decision: "Beginner books avoid discussing OPTIONS and PATCH methods"
    rationale: "GET, POST, PUT, DELETE cover 95% of use cases. OPTIONS and PATCH add complexity without proportional value for beginners"
    alternatives_considered: ["Cover all methods", "Mention briefly"]
    impact: "book_001 doesn't cover OPTIONS/PATCH. book_002 introduces them"
    status: "approved"
```

---

## 10. Change History

### 10.1 Terminology Changes

**Purpose**: Track what changed and when

```yaml
terminology_changes:
  - change_id: "change_001"
    date: "2024-05-01"
    old_term: "API call"
    new_term: "API request"
    reason: "More technically precise"
    books_affected: ["book_002", "book_003"]
    migration_plan: "Update in next editions"
  
  - change_id: "change_002"
    date: "2024-05-02"
    old_definition: "REST is a style for designing APIs"
    new_definition: "REST is an architectural style for distributed systems"
    reason: "More accurate, not limited to just 'design'"
    books_affected: ["book_001"]
    migration_plan: "Update book_001 v1.1"
```

### 10.2 Concept Evolution

**Purpose**: Track how explanations improve

```yaml
concept_evolution:
  - concept: "Authentication vs Authorization"
    version_1: "Authentication checks who you are. Authorization checks what you can do."
    version_2: "Authentication verifies identity (who you are). Authorization determines permissions (what you're allowed to access)."
    change_date: "2024-05-03"
    reason: "More precise language, clearer distinction"
    books_using_v2: ["book_002", "book_003"]
```

---

## 11. Cross-References

### 11.1 Inter-Book References

**Purpose**: Track how books reference each other

```yaml
cross_references:
  - from_book: "book_002"
    to_book: "book_001"
    reference: "For an introduction to REST principles, see REST APIs for Beginners, Chapter 1"
    context: "book_002 assumes book_001 knowledge"
  
  - from_book: "book_003"
    to_book: "book_002"
    reference: "Advanced design patterns are covered in Advanced REST API Design, Chapter 5"
    context: "book_003 extends book_002"
```

---

## 12. Quality Metrics

### 12.1 Consistency Scores

**Purpose**: Measure cross-book consistency

```yaml
consistency_metrics:
  - metric: "term_consistency"
    measurement: "Percentage of shared terms using canonical form"
    target: "95%"
    current: "92%"
    date_measured: "2024-05-05"
  
  - metric: "definition_alignment"
    measurement: "Percentage of concepts with matching definitions"
    target: "100%"
    current: "98%"
    date_measured: "2024-05-05"
```

---

## 13. Maintenance Plan

### 13.1 Review Schedule

**Purpose**: Keep continuity bible up to date

```yaml
maintenance:
  review_frequency: "quarterly"
  last_review: "2024-05-05"
  next_review: "2024-08-05"
  responsibilities:
    - role: "Content lead"
      tasks: ["Review terminology changes", "Update decision log"]
    - role: "Technical reviewer"
      tasks: ["Verify technical accuracy", "Update examples"]
```

---

## 14. Template for New Books

### 14.1 Pre-Production Checklist

**Before starting a new book**:

- [ ] Review canonical terminology list
- [ ] Check which concepts will be reused
- [ ] Identify reader prerequisites
- [ ] Plan knowledge progression
- [ ] Note any terminology conflicts
- [ ] Review standard examples available
- [ ] Check visual conventions
- [ ] Record new decisions in decision log

### 14.2 Post-Production Updates

**After completing a new book**:

- [ ] Add new canonical terms to master list
- [ ] Document new examples in library
- [ ] Record new analogies (if effective)
- [ ] Update knowledge progression map
- [ ] Log editorial decisions
- [ ] Update cross-references
- [ ] Measure consistency metrics

---

## Example Continuity Bible Entry

```yaml
continuity_bible:
  book_series: "REST API Mastery Series"
  
  canonical_terms:
    - term: "REST API"
      definition: "An API following REST architectural principles"
      aliases: ["RESTful API", "RESTful service"]
      note: "Always use 'REST API'"
  
  recurring_concepts:
    - concept: "Statelessness"
      explanation: "Each request contains all needed information"
      analogy: "Restaurant waiter—each order complete"
      books: ["book_001_ch02", "book_002_ch01"]
  
  standard_examples:
    - id: "ex_001"
      title: "Basic CRUD"
      code: "GET/POST/PUT/DELETE /users"
      books: ["book_001", "book_002"]
  
  decisions:
    - id: "dec_001"
      decision: "Cover only GET/POST/PUT/DELETE in Book 1"
      date: "2024-05-01"
      rationale: "Simplify for beginners"
  
  change_history:
    - change: "Renamed 'API call' to 'API request'"
      date: "2024-05-02"
      reason: "More precise"
```

---

## Version History

- **v1.0** (2024-05-05): Initial continuity bible template
