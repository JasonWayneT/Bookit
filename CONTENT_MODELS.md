# Content Models: Structured Metadata Schemas

## Document Control

**Version**: 1.0  
**Purpose**: Define structured metadata for all system entities  
**Scope**: Source items, books, chapters, glossary, visuals, validation  
**Status**: Active  

---

## 1. Source Item Model

```yaml
source_item:
  # Identification
  id: uuid  # e.g., "src_a3f9c2d1"
  filename: string  # e.g., "meeting_transcript.txt"
  file_hash: string  # SHA-256 for deduplication
  
  # Classification
  content_type: enum
    values: [transcript, notes, article, research, ai_chat, slides, technical_doc, dataset, image]
  classification_confidence: float  # 0.0-1.0
  classification_method: string  # e.g., "AI classification v1.0"
  
  # Content
  raw_text: text  # Original extracted text
  file_size_bytes: integer
  word_count: integer
  encoding: string  # e.g., "UTF-8"
  language: string  # e.g., "en-US"
  
  # Extraction metadata
  extraction_method: string  # e.g., "pdfplumber", "python-docx"
  extraction_quality: enum [high, medium, low]
  extraction_issues: array[string]  # Warnings or errors
  
  # Processing status
  status: enum [uploaded, classified, processed, validated, archived]
  processing_started: datetime
  processing_completed: datetime
  retry_count: integer
  
  # Source tracking
  original_url: string (optional)  # If downloaded from web
  upload_date: datetime
  uploaded_by: string
  
  # Content analysis
  topic_keywords: array[string]  # Extracted topics
  detected_terms: array[string]  # Technical terms found
  quality_score: float  # 0.0-1.0
  
  # Relationships
  derived_books: array[book_id]  # Books generated from this source
  combined_with_sources: array[source_id]  # If merged with other sources
  
  # Metadata
  created_date: datetime
  last_modified: datetime
  notes: text
```

**Example**:

```yaml
source_item:
  id: "src_001"
  filename: "rest_api_lecture.txt"
  file_hash: "a1b2c3..."
  content_type: transcript
  classification_confidence: 0.95
  classification_method: "AI classification v1.0"
  raw_text: "So, um, REST stands for..."
  file_size_bytes: 45000
  word_count: 7500
  encoding: "UTF-8"
  language: "en-US"
  extraction_method: "direct text read"
  extraction_quality: high
  extraction_issues: []
  status: processed
  processing_started: "2024-05-05T10:00:00Z"
  processing_completed: "2024-05-05T10:45:00Z"
  retry_count: 0
  original_url: null
  upload_date: "2024-05-05T09:30:00Z"
  uploaded_by: "user_123"
  topic_keywords: ["REST", "API", "HTTP", "web services"]
  detected_terms: ["REST", "API", "HTTP method", "stateless"]
  quality_score: 0.85
  derived_books: ["book_001"]
  combined_with_sources: []
  created_date: "2024-05-05T09:30:00Z"
  last_modified: "2024-05-05T10:45:00Z"
  notes: "Lecture from REST API course, recorded April 2024"
```

---

## 2. Book Model

```yaml
book:
  # Identification
  id: uuid
  title: string
  subtitle: string (optional)
  version: string  # e.g., "1.0", "1.1"
  
  # Promise
  audience_level: enum [beginner, practitioner, advanced, expert]
  transformation_goal: text  # What reader will be able to do
  scope_included: array[string]  # Topics covered
  scope_excluded: array[string]  # Explicitly not covered
  payoff: text  # Why this matters
  
  # Structure
  uses_parts: boolean
  part_count: integer
  chapter_count: integer
  section_count: integer
  estimated_pages: integer
  estimated_reading_time_minutes: integer
  
  # Content metrics
  word_count: integer
  term_count: integer  # Unique terms defined
  example_count: integer
  visual_count: integer
  code_block_count: integer
  
  # Progression
  progression_pattern: enum
    values: [beginner_to_advanced, problem_to_solution, concept_to_application, history_to_current, overview_to_components]
  
  # Source tracking
  source_item_ids: array[uuid]
  primary_source_id: uuid  # Main source if multiple
  
  # Processing status
  status: enum [draft, validated, published, archived]
  current_stage: enum [intake, structure, expansion, visual, validation, output]
  validation_score: float  # 0.0-1.0
  validation_issues: array[object]
  
  # Guide file versions used
  guide_files_used:
    content_guide_version: string
    style_manifest_version: string
    architecture_version: string
    pedagogy_version: string
    vocabulary_version: string
    visual_version: string
    taxonomy_version: string
  
  # Relationships
  part_ids: array[uuid]
  chapter_ids: array[uuid]
  glossary_term_ids: array[uuid]
  visual_ids: array[uuid]
  related_books: array[uuid]  # Other books in series
  
  # Publishing
  published_date: datetime (optional)
  published_formats: array[enum [html, pdf, epub]]
  public_url: string (optional)
  download_count: integer
  
  # Metadata
  created_date: datetime
  last_modified: datetime
  created_by: string  # "AI system v1.0"
  notes: text
```

---

## 3. Part Model

```yaml
part:
  # Identification
  id: uuid
  book_id: uuid
  part_number: integer
  
  # Content
  title: string
  introduction: text  # 100-300 words
  themes: array[string]  # Major themes in this part
  
  # Structure
  chapter_ids: array[uuid]
  chapter_count: integer
  
  # Metrics
  word_count: integer
  estimated_pages: integer
  estimated_reading_time_minutes: integer
  
  # Metadata
  created_date: datetime
  last_modified: datetime
```

---

## 4. Chapter Model

```yaml
chapter:
  # Identification
  id: uuid
  book_id: uuid
  part_id: uuid (optional)
  chapter_number: integer
  
  # Content
  title: string
  teaching_objective: string  # One primary goal
  opening_hook: text  # Engaging intro
  core_content: text  # Main explanatory text
  chapter_recap: text  # Summary of key points
  bridge_to_next: text  # Transition to next chapter
  
  # Prerequisites
  prerequisite_concepts: array[string]
  prerequisite_chapters: array[chapter_id]
  
  # Structure
  section_ids: array[uuid]
  section_count: integer
  subsection_count: integer
  
  # Content classification
  reader_level: enum [beginner, practitioner, advanced]
  content_role_distribution: object
    # e.g., {DEFINITION: 10, EXAMPLE: 15, MECHANISM: 8}
  
  # Terms
  terms_introduced: array[string]
  terms_used: array[string]
  first_use_terms: array[string]
  
  # Examples and visuals
  example_count: integer
  example_ids: array[uuid]
  visual_count: integer
  visual_ids: array[uuid]
  code_block_count: integer
  
  # Source mapping
  source_sections: array[object]
    # {source_id, start_line, end_line, transformed_to_section_id}
  
  # Metrics
  word_count: integer
  estimated_reading_time_minutes: integer
  complexity_score: float  # 0.0-1.0 (0=simple, 1=complex)
  
  # Validation
  validation_results: object
    pedagogical_score: float
    style_score: float
    completeness_score: float
    issues: array[string]
  
  # Status
  status: enum [drafted, expanded, illustrated, reviewed, validated]
  completed: boolean
  needs_revision: boolean
  revision_notes: text
  
  # Metadata
  created_date: datetime
  last_modified: datetime
  version: integer
```

---

## 5. Section Model

```yaml
section:
  # Identification
  id: uuid
  chapter_id: uuid
  section_number: string  # e.g., "2.1", "2.1.1"
  
  # Content
  title: string
  content: text
  supporting_concept: string  # One concept this section teaches
  
  # Classification
  content_roles: array[enum]  # [DEFINITION, MECHANISM, EXAMPLE, etc.]
  reader_level: enum [beginner, practitioner, advanced]
  
  # Structure
  subsection_ids: array[uuid]
  block_count: integer
  
  # Terms and examples
  terms_introduced: array[string]
  example_count: integer
  visual_count: integer
  
  # Metrics
  word_count: integer
  estimated_reading_time_minutes: integer
  
  # Metadata
  created_date: datetime
  last_modified: datetime
```

---

## 6. Glossary Term Model

```yaml
glossary_term:
  # Identification
  id: uuid
  term: string  # Canonical form
  term_lowercase: string  # For case-insensitive search
  aliases: array[string]  # Alternative names
  acronym_full_form: string (optional)
  
  # Definitions
  plain_definition: text  # Beginner-friendly
  technical_definition: text (optional)  # Precise, formal
  
  # Examples
  example: text (optional)
  non_example: text (optional)
  code_example: text (optional)
  
  # Context
  domain: string  # e.g., "web development"
  category: enum
    values: [concept, method, tool, pattern, protocol, data_structure]
  
  # Usage tracking
  first_appearance:
    chapter_id: uuid
    chapter_number: integer
    section_title: string
    inline_definition: text
  subsequent_uses: array[chapter_id]
  usage_count: integer
  
  # Relationships
  related_terms: array[term_id]
  prerequisite_terms: array[term_id]
  broader_term: term_id (optional)
  narrower_terms: array[term_id]
  synonyms: array[term_id]
  antonyms: array[term_id]
  
  # Status
  status: enum [draft, reviewed, approved]
  verified: boolean
  verification_source: string (optional)
  
  # Metadata
  created_date: datetime
  last_updated: datetime
  created_by: string
```

---

## 7. Visual Asset Model

```yaml
visual:
  # Identification
  id: uuid
  figure_number: string  # e.g., "3.2"
  type: enum
    values: [diagram, chart, callout, timeline, table, screenshot, code_viz]
  subtype: string (optional)  # e.g., "bar_chart", "flow_diagram"
  
  # Content
  title: string
  caption: text
  alt_text: text
  description: text  # Detailed description for metadata
  
  # Generation
  generation_method: enum
    values: [ai_svg, ai_image_api, visualization_library, manual, extracted]
  creation_prompt: text (optional)  # AI prompt if AI-generated
  data_source: string (optional)  # "Real benchmark" or "Illustrative"
  data_points: object (optional)  # Structured data for charts
  
  # Location
  chapter_id: uuid
  section_id: uuid (optional)
  placement: enum [inline, full_width, margin, appendix]
  position_in_chapter: integer  # Sequence number
  
  # Source
  source_content_ids: array[string]  # Text sections this illustrates
  illustrates_concepts: array[string]  # What concepts it shows
  
  # Technical
  file_path: string
  file_format: enum [svg, png, jpg, webp, gif]
  dimensions:
    width: integer
    height: integer
  file_size_bytes: integer
  resolution_dpi: integer (optional)
  
  # Accessibility
  color_blind_safe: boolean
  screen_reader_compatible: boolean
  text_alternatives_provided: boolean
  
  # Status
  status: enum [planned, generated, reviewed, approved, needs_update]
  needs_update: boolean
  update_reason: text
  version: integer
  
  # Metadata
  created_date: datetime
  last_updated: datetime
  created_by: string
  review_notes: text
```

---

## 8. Citation Model

```yaml
citation:
  # Identification
  id: uuid
  citation_key: string  # e.g., "fielding_2000"
  
  # Source information
  type: enum
    values: [specification, research_paper, book, article, blog_post, documentation, benchmark]
  
  title: string
  authors: array[string]
  publication_date: string  # YYYY-MM-DD or YYYY
  publisher: string (optional)
  url: string (optional)
  doi: string (optional)
  
  # Content
  abstract: text (optional)
  key_findings: text (optional)
  relevance: text  # Why this source matters for the book
  
  # Usage
  cited_in_chapters: array[chapter_id]
  citation_count: integer
  quote_excerpts: array[text]  # Specific quotes used
  
  # Classification
  evidence_type: enum
    values: [primary_source, secondary_source, anecdote, data, interpretation, hypothesis]
  confidence_level: enum [high, medium, low]
  
  # Status
  verified: boolean
  verification_date: datetime (optional)
  access_date: datetime  # When source was last accessed
  
  # Metadata
  created_date: datetime
  last_updated: datetime
```

---

## 9. Validation Record Model

```yaml
validation_record:
  # Identification
  id: uuid
  book_id: uuid
  chapter_id: uuid (optional)  # If chapter-level validation
  validation_timestamp: datetime
  
  # Overall results
  overall_score: float  # 0.0-1.0
  passed: boolean  # True if score >= threshold (e.g., 0.8)
  
  # Category scores
  pedagogical_score: float
  style_score: float
  completeness_score: float
  attribution_score: float
  technical_accuracy_score: float (optional)
  
  # Checks performed
  checks_run: integer
  checks_passed: integer
  checks_failed: integer
  checks_warning: integer
  
  # Detailed results
  pedagogical_checks: array[check_result]
  style_checks: array[check_result]
  completeness_checks: array[check_result]
  attribution_checks: array[check_result]
  
  # Issues found
  critical_issues: array[issue]
  warnings: array[issue]
  suggestions: array[issue]
  
  # Validation configuration
  validation_rules_version: string
  thresholds: object
    pass_threshold: float
    warning_threshold: float
  
  # Status
  requires_revision: boolean
  revision_priority: enum [high, medium, low]
  assigned_to: string (optional)
  
  # Follow-up
  resolved: boolean
  resolution_date: datetime (optional)
  resolution_notes: text
  
  # Metadata
  validated_by: string  # "AI validation system v1.0"
  created_date: datetime
```

**Check Result Schema**:

```yaml
check_result:
  check_name: string
  check_description: text
  category: enum [pedagogical, style, completeness, attribution]
  passed: boolean
  score: float (optional)  # If scored check
  severity: enum [critical, warning, info]
  details: text
  affected_locations: array[object]
    # {chapter_id, section_id, line_range, excerpt}
  recommendation: text  # How to fix
```

**Issue Schema**:

```yaml
issue:
  type: string  # e.g., "undefined_term", "missing_example"
  severity: enum [critical, warning, info]
  location:
    chapter_id: uuid
    section_id: uuid (optional)
    line_number: integer (optional)
  description: text
  excerpt: text  # Relevant text snippet
  recommendation: text
  auto_fixable: boolean
```

---

## 10. Editorial Decision Model

```yaml
editorial_decision:
  # Identification
  id: uuid
  decision_type: enum
    values: [structure_choice, content_inclusion, tone_adjustment, visual_selection, term_choice]
  
  # Context
  book_id: uuid
  chapter_id: uuid (optional)
  section_id: uuid (optional)
  
  # Decision
  decision: text  # What was decided
  rationale: text  # Why this decision was made
  alternatives_considered: array[text]
  trade_offs: object
    gains: array[string]
    losses: array[string]
  
  # Impact
  affects: array[string]  # What this impacts
  prerequisite_decisions: array[decision_id]
  dependent_decisions: array[decision_id]
  
  # Attribution
  decided_by: string  # "AI system" or "human reviewer"
  decision_date: datetime
  
  # Tracking
  status: enum [proposed, approved, implemented, reversed]
  reversed_by: decision_id (optional)
  reversal_reason: text (optional)
  
  # Metadata
  created_date: datetime
  last_modified: datetime
```

---

## 11. Processing Job Model

```yaml
processing_job:
  # Identification
  id: uuid
  job_type: enum
    values: [intake, classification, structure, expansion, visual_generation, validation, output_generation]
  
  # Target
  book_id: uuid
  chapter_id: uuid (optional)
  source_item_id: uuid (optional)
  
  # Status
  status: enum [queued, in_progress, completed, failed, cancelled]
  progress_percentage: float  # 0.0-100.0
  current_step: string
  total_steps: integer
  
  # Execution
  started_at: datetime (optional)
  completed_at: datetime (optional)
  duration_seconds: integer (optional)
  retry_count: integer
  max_retries: integer
  
  # Results
  output: object  # Job-specific output
  errors: array[object]
  warnings: array[object]
  
  # Resources
  ai_model_used: string  # e.g., "claude-sonnet-4"
  ai_tokens_used: integer
  ai_cost_usd: float
  
  # Metadata
  created_date: datetime
  created_by: string
  log_file: string  # Path to detailed log
```

---

## 12. Version History

- **v1.0** (2024-05-05): Initial content models
