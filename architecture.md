# Book System: Architecture Document

## Document Control

**Version**: 1.0  
**Date**: 2024-05-05  
**Status**: Draft  
**Owner**: Architecture Team  
**Reviewers**: Engineering, Product, AI/ML  

## 1. Architecture Overview

### 1.1 System Purpose

Book System is an AI-first content transformation pipeline that converts raw non-fiction materials into publication-ready vertical visual reports through automated classification, expansion, visual enhancement, and validation.

### 1.2 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        User Interface Layer                      │
│  (Upload Interface, Configuration, Preview, Download)            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                      Orchestration Layer                         │
│        (Pipeline Controller, State Manager, Job Queue)           │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                    Processing Pipeline Stages                    │
│                                                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   Intake    │→ │  Structure  │→ │  Expansion  │             │
│  │   Engine    │  │   Engine    │  │   Engine    │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                                                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   Visual    │→ │ Validation  │→ │   Output    │             │
│  │   Engine    │  │   Engine    │  │   Engine    │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                      Foundation Services                         │
│  (AI API, Storage, Metadata DB, Guide File Repository)          │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 Design Principles

**ARCH-001**: AI-First Processing  
All content transformation is performed by AI with minimal human intervention. Humans provide inputs and review outputs, but do not participate in the transformation process.

**ARCH-002**: Guide-File Driven  
All transformation rules, style decisions, and validation criteria are defined in versioned guide files (markdown documents) that the AI reads before processing. Changes to behavior happen by updating guides, not code.

**ARCH-003**: Stateless Pipeline Stages  
Each processing stage is stateless and idempotent. State is maintained in metadata storage, not in stage components. This enables retry, parallelization, and debugging.

**ARCH-004**: Progressive Enhancement  
The system produces a valid output at each stage. Early stages produce simple but complete books; later stages add sophistication. A failure in visual generation doesn't block publication.

**ARCH-005**: Metadata-Rich  
Every transformation decision, source attribution, and quality check is recorded as metadata. The system can explain why it made any decision.

**ARCH-006**: Vertical Visual Output  
All output formatting optimizes for digital scrolling, not paginated reading. Layout, typography, and visual placement assume screen consumption.

## 2. Component Architecture

### 2.1 Intake Engine

**ARCH-007**: Component Purpose  
Accept raw content files, extract text, classify content type, and create source metadata.

**Responsibilities**:
- File upload handling (.txt, .md, .pdf, .docx, .srt)
- Text extraction from various formats
- Content type classification (transcript, notes, article, slides, technical doc)
- Source metadata creation
- Raw content preservation

**Interfaces**:
- **Input**: User file upload, file path reference
- **Output**: `source_item` metadata object, extracted text
- **Storage**: Raw files → object storage, metadata → database

**Key Algorithms**:
- **Classification**: AI prompt with file content sample + format hints → content type
- **Text Extraction**: 
  - PDF: PyPDF2 or pdfplumber for text extraction
  - DOCX: python-docx for structure-aware extraction
  - SRT: Custom parser for timestamp/speaker/text
  - TXT/MD: Direct read with encoding detection

**Dependencies**:
- File format libraries (PyPDF2, python-docx, chardet)
- AI API for classification
- Object storage (S3 or equivalent)
- Metadata database

**Configuration**:
Reads from `SOURCE_INPUT_TYPES.md` guide file for:
- Content type definitions
- Classification criteria
- Processing rules per type
- Metadata requirements

**Error Handling**:
- Malformed files: Flag for manual review, attempt partial extraction
- Unsupported formats: Reject with clear error message
- Extraction failures: Preserve raw file, log diagnostic info

**ARCH-008**: Intake Metadata Schema
```yaml
source_item:
  id: uuid
  filename: string
  content_type: enum
  upload_timestamp: datetime
  file_size_bytes: integer
  content_hash: string
  extracted_text: text
  classification_confidence: float
  processing_status: enum
  error_log: array[string]
```

### 2.2 Structure Engine

**ARCH-009**: Component Purpose  
Analyze content, build book hierarchy, detect prerequisites, identify gaps, and create organizational structure.

**Responsibilities**:
- Theme detection and Part creation
- Chapter segmentation based on teaching objectives
- Section and subsection breakdown
- Prerequisite relationship mapping
- Gap detection (missing explanations, assumed knowledge)
- Sequencing recommendation

**Interfaces**:
- **Input**: `source_item` with extracted text
- **Output**: `book` metadata with hierarchy, `chapter` objects, gap report
- **Storage**: Hierarchy metadata → database

**Key Algorithms**:
- **Hierarchy Generation**: 
  - AI prompt with full text + BOOK_ARCHITECTURE.md rules
  - Output: JSON structure of book → parts → chapters → sections
  - Validation: Ensure single teaching objective per chapter
  
- **Prerequisite Detection**:
  - AI analyzes each chapter for assumed concepts
  - Cross-reference with earlier chapters
  - Flag concepts used before definition
  
- **Gap Detection**:
  - Compare chapters against PEDAGOGY_RULES.md
  - Identify missing scaffolding
  - Detect unexplained jargon

**Dependencies**:
- AI API for structure analysis
- Guide files: BOOK_ARCHITECTURE.md, PEDAGOGY_RULES.md, BOOK_TAXONOMY.md

**Configuration**:
Reads from guide files for:
- Hierarchy models (part/chapter/section definitions)
- Sequencing patterns (beginner→advanced, problem→solution)
- Gap detection rules

**Performance**:
- Process 100-page document in < 5 minutes
- Generate complete hierarchy with 80%+ accuracy
- Detect 90%+ of prerequisite violations

**ARCH-010**: Structure Metadata Schema
```yaml
book:
  id: uuid
  title: string
  structure_version: integer
  parts: array[part_object]
  chapters: array[chapter_id]
  prerequisite_graph: adjacency_list
  detected_gaps: array[gap_object]
  structure_confidence: float

chapter:
  id: uuid
  number: integer
  title: string
  teaching_objective: string
  prerequisite_chapters: array[chapter_id]
  prerequisite_concepts: array[string]
  sections: array[section_object]
  estimated_length: integer
  source_mapping: array[source_ref]
```

### 2.3 Expansion Engine

**ARCH-011**: Component Purpose  
Transform raw content into beginner-accessible prose with scaffolding, definitions, examples, and pedagogical enhancements.

**Responsibilities**:
- Detect technical terms and jargon
- Generate plain-English definitions
- Add beginner scaffolding for advanced concepts
- Create concrete examples
- Insert analogies where helpful
- Add chapter recaps and bridges
- Detect and address misconceptions

**Interfaces**:
- **Input**: `chapter` objects with raw content
- **Output**: Expanded chapter text, `glossary_term` objects
- **Storage**: Expanded text → database, glossary → dedicated collection

**Key Algorithms**:
- **Term Detection**:
  - AI analyzes chapter with VOCABULARY_AND_GLOSSARY_MODEL.md rules
  - Extracts technical terms, acronyms, domain vocabulary
  - Output: List of terms with context
  
- **Definition Generation**:
  - For each term: AI generates plain + technical definition
  - Adds example and non-example
  - Inserts inline on first use
  
- **Scaffolding Insertion**:
  - AI detects complexity jumps
  - Adds "what you need to know" sections
  - Provides prerequisite refreshers
  
- **Example Creation**:
  - AI generates concrete example after abstract explanation
  - Ensures example matches concept complexity
  - Uses realistic, relatable scenarios

**Dependencies**:
- AI API for content generation
- Guide files: VOCABULARY_AND_GLOSSARY_MODEL.md, PEDAGOGY_RULES.md, BOOK_STYLE_MANIFEST.md
- Glossary database for term reuse

**Configuration**:
Reads from guide files for:
- Term detection criteria
- Definition level requirements
- Scaffolding triggers
- Example generation rules
- Style and tone guidelines

**Quality Metrics**:
- 90%+ of technical terms defined on first use
- Average 1 example per major concept
- Scaffolding added for 80%+ of complexity jumps

**ARCH-012**: Glossary Metadata Schema
```yaml
glossary_term:
  id: uuid
  term: string
  canonical_form: string
  aliases: array[string]
  plain_definition: text
  technical_definition: text
  example: text
  non_example: text
  first_appearance_chapter_id: uuid
  related_terms: array[term_id]
  usage_count: integer
  created_timestamp: datetime
```

### 2.4 Visual Engine

**ARCH-013**: Component Purpose  
Identify visual candidates, generate diagrams and charts, create callouts, and produce visual assets for the book.

**Responsibilities**:
- Identify where visuals would enhance understanding
- Generate diagrams from process/system descriptions
- Create charts from data mentions
- Design callout boxes for key concepts
- Add captions and alt text
- Store visual metadata

**Interfaces**:
- **Input**: Expanded chapter text
- **Output**: `visual` metadata objects, image files
- **Storage**: Images → object storage, metadata → database

**Key Algorithms**:
- **Visual Candidate Detection**:
  - AI analyzes text with VISUAL_CONTENT_MODEL.md rules
  - Detects: process descriptions, comparisons, system architecture, timelines
  - Output: Visual specification (type, content, location)
  
- **Diagram Generation**:
  - Option A: AI generates SVG/HTML directly
  - Option B: AI creates description → image generation API
  - Option C: AI uses Mermaid/PlantUML syntax
  - Fallback: Text-based representation
  
- **Chart Creation**:
  - Extract numerical data from text
  - Determine appropriate chart type (bar, line, pie)
  - Generate chart using visualization library
  - Add labels, title, legend
  
- **Callout Generation**:
  - Identify key definitions, warnings, tips
  - Extract text for callout
  - Format with consistent styling

**Dependencies**:
- AI API for visual planning
- Image generation API (optional)
- Visualization library (e.g., D3.js, Chart.js)
- Guide file: VISUAL_CONTENT_MODEL.md

**Configuration**:
Reads from guide file for:
- Visual selection criteria
- Visual type definitions
- Chart styling rules
- Caption and alt text requirements

**Performance**:
- Generate visuals asynchronously (non-blocking)
- Cache generated images
- Provide placeholders during generation
- Target: 1 visual per 500-1000 words

**ARCH-014**: Visual Metadata Schema
```yaml
visual:
  id: uuid
  type: enum[diagram, chart, callout, timeline, table, screenshot]
  title: string
  caption: text
  alt_text: text
  source_chapter_id: uuid
  source_content_refs: array[string]
  generation_method: enum[ai_svg, api_image, library_chart, manual]
  file_path: string
  file_format: enum[svg, png, jpg]
  dimensions: {width: integer, height: integer}
  created_timestamp: datetime
  generation_metadata: object
```

### 2.5 Validation Engine

**ARCH-015**: Component Purpose  
Run automated quality checks against the transformed content to ensure it meets pedagogical, style, and completeness standards.

**Responsibilities**:
- Execute pedagogical validation (terms defined, examples present, scaffolding added)
- Check style consistency (tone, terminology, voice)
- Verify content completeness (all chapters have required elements)
- Validate source attribution (content traces to sources)
- Generate validation report with pass/fail for each check

**Interfaces**:
- **Input**: Complete book with all chapters, glossary, visuals
- **Output**: `validation_report` with scores and failures
- **Storage**: Report → database, linked to book version

**Key Algorithms**:
- **Pedagogical Validation**:
  - Check: All terms in glossary appear on first use with definition
  - Check: Every abstract concept followed by example
  - Check: Advanced concepts have scaffolding
  - Scoring: Pass/fail per check, aggregate score
  
- **Style Validation**:
  - Check: Term consistency (same term used throughout, no alternating synonyms)
  - Check: Tone matches BOOK_STYLE_MANIFEST.md
  - Check: No forbidden patterns (jargon stacking, unexplained acronyms)
  - Scoring: Pattern match count, threshold-based pass/fail
  
- **Completeness Validation**:
  - Check: All chapters have teaching objective, examples, recap
  - Check: Visual-to-text ratio meets targets
  - Check: All glossary terms referenced in text
  - Scoring: Count missing elements
  
- **Attribution Validation**:
  - Check: All content maps to source items
  - Check: No unsupported factual claims
  - Scoring: Coverage percentage

**Dependencies**:
- Guide files: VALIDATION_CHECKLISTS.md, PEDAGOGY_RULES.md, BOOK_STYLE_MANIFEST.md
- Book metadata with full content
- Glossary and visual metadata

**Configuration**:
Reads from VALIDATION_CHECKLISTS.md for:
- Complete list of checks per category
- Pass/fail criteria for each check
- Scoring weights

**Quality Target**:
- 80% automated validation pass rate
- <20% false positives (flagged issues that are actually fine)
- Generate report in < 5 minutes for 100-page book

**ARCH-016**: Validation Report Schema
```yaml
validation_report:
  id: uuid
  book_id: uuid
  validation_timestamp: datetime
  overall_score: float
  passed: boolean
  checks_run: integer
  checks_passed: integer
  checks_failed: integer
  
  pedagogical_results:
    score: float
    checks: array[check_result]
    
  style_results:
    score: float
    checks: array[check_result]
    
  completeness_results:
    score: float
    checks: array[check_result]
    
  attribution_results:
    score: float
    checks: array[check_result]

check_result:
  check_name: string
  passed: boolean
  details: text
  severity: enum[critical, warning, info]
  affected_chapters: array[chapter_id]
```

### 2.6 Output Engine

**ARCH-017**: Component Purpose  
Render the final vertical visual report in responsive HTML and export to PDF, applying consistent design system and optimizing for digital reading.

**Responsibilities**:
- Apply design system (typography, colors, spacing)
- Render responsive HTML optimized for vertical scrolling
- Generate table of contents with navigation
- Embed visuals with proper placement
- Export to PDF while maintaining formatting
- Bundle downloadable assets

**Interfaces**:
- **Input**: Validated book with all content, glossary, visuals
- **Output**: HTML document, PDF file, asset bundle
- **Storage**: HTML/PDF → object storage, URL → database

**Key Algorithms**:
- **HTML Rendering**:
  - Apply design system CSS (responsive, mobile-first)
  - Progressive disclosure: content revealed on scroll
  - Sticky navigation for table of contents
  - Lazy-load images for performance
  
- **PDF Export**:
  - Use headless browser (Puppeteer) or PDF library
  - Preserve hyperlinks and navigation
  - Generate bookmarks from table of contents
  - Maintain visual formatting
  
- **Asset Bundling**:
  - Collect all images, diagrams, code samples
  - Create downloadable ZIP
  - Generate manifest

**Dependencies**:
- HTML/CSS framework (Tailwind, custom design system)
- PDF rendering library (Puppeteer, WeasyPrint, or similar)
- Asset storage service

**Configuration**:
- Design system variables defined in guide files or CSS
- Layout templates for different content types
- Export settings (page size, margins, quality)

**Performance**:
- Render HTML in < 30 seconds for 100-page book
- Export PDF in < 2 minutes
- Optimize images for web delivery
- Support responsive breakpoints: 320px, 768px, 1024px, 1440px

**ARCH-018**: Output Metadata Schema
```yaml
output:
  id: uuid
  book_id: uuid
  format: enum[html, pdf, epub]
  file_path: string
  file_size_bytes: integer
  generated_timestamp: datetime
  design_version: string
  asset_count: integer
  asset_bundle_path: string
  public_url: string
  download_count: integer
```

## 3. Data Flow

### 3.1 Primary Processing Flow

```
1. User uploads raw content file
   ↓
2. Intake Engine:
   - Extracts text
   - Classifies content type
   - Creates source_item metadata
   ↓
3. Structure Engine:
   - Analyzes content
   - Builds hierarchy (parts/chapters/sections)
   - Detects prerequisites and gaps
   - Creates book and chapter metadata
   ↓
4. Expansion Engine:
   - Detects technical terms
   - Generates definitions and glossary
   - Adds scaffolding and examples
   - Expands raw content to teaching prose
   - Updates chapter content
   ↓
5. Visual Engine (parallel to expansion):
   - Identifies visual candidates
   - Generates diagrams and charts
   - Creates callouts
   - Stores visual metadata
   ↓
6. Validation Engine:
   - Runs pedagogical checks
   - Validates style and completeness
   - Checks attribution
   - Generates validation report
   ↓
7. Output Engine:
   - Renders HTML with design system
   - Exports to PDF
   - Bundles assets
   - Publishes output
   ↓
8. User reviews and downloads
```

### 3.2 Guide File Access Pattern

At each stage, the processing engine:
1. Identifies relevant guide file(s) for the current operation
2. Reads guide file from repository (cached if unchanged)
3. Includes guide content in AI prompt as instructions
4. Executes transformation based on guide rules
5. Logs which guide version was used

**Example** (Expansion Engine):
```
1. Load VOCABULARY_AND_GLOSSARY_MODEL.md (v1.2)
2. Load PEDAGOGY_RULES.md (v1.1)
3. Load BOOK_STYLE_MANIFEST.md (v1.0)
4. Construct AI prompt:
   - Chapter raw text
   - Guide file instructions
   - Previous glossary terms (for consistency)
5. AI generates expanded chapter
6. Store result + metadata (guide versions used)
```

### 3.3 State Management

**ARCH-019**: Processing State Tracking

Each book maintains a processing state:
```yaml
processing_state:
  book_id: uuid
  current_stage: enum[intake, structure, expansion, visual, validation, output]
  stage_status: enum[pending, in_progress, completed, failed]
  started_timestamp: datetime
  completed_timestamp: datetime
  retry_count: integer
  error_log: array[error_object]
  checkpoint_data: object
```

**Checkpoint Strategy**:
- After each stage completes, save checkpoint
- If stage fails, retry from last checkpoint
- If retry fails 3 times, mark for manual review
- Checkpoints enable resuming long-running jobs

## 4. Technology Stack

### 4.1 Core Technologies

**ARCH-020**: Language and Runtime
- **Primary Language**: Python 3.11+
- **Rationale**: Rich ecosystem for text processing, AI APIs, PDF/DOCX handling
- **Alternative**: Node.js for web-focused components

**ARCH-021**: AI Integration
- **Primary**: Anthropic Claude API (Claude Opus or Sonnet)
- **Rationale**: Strong long-context capability, instruction following, structured output
- **Fallback**: OpenAI GPT-4 API
- **Consideration**: Local models (Llama 3) for cost/privacy-sensitive deployments

**ARCH-022**: Storage
- **Object Storage**: AWS S3 or compatible (MinIO for self-hosted)
- **Metadata Database**: PostgreSQL 15+
- **Rationale**: Structured metadata with relationships, full-text search
- **Alternative**: MongoDB for more flexible schema

**ARCH-023**: Processing Framework
- **Queue System**: Redis + Celery for async task processing
- **Rationale**: Enables parallel stage execution, retry logic, monitoring
- **Alternative**: AWS SQS + Lambda for serverless

**ARCH-024**: Output Rendering
- **HTML/CSS**: Tailwind CSS + custom design system
- **PDF Generation**: Puppeteer (headless Chrome)
- **Rationale**: High-fidelity PDF from HTML, maintains interactivity
- **Alternative**: WeasyPrint for server-side rendering

### 4.2 Libraries and Dependencies

**Text Processing**:
- PyPDF2 or pdfplumber (PDF extraction)
- python-docx (DOCX extraction)
- chardet (encoding detection)
- beautifulsoup4 (HTML parsing if needed)

**AI Integration**:
- anthropic (Claude API client)
- openai (GPT API client, fallback)
- tiktoken (token counting)

**Visualization**:
- Pillow (image manipulation)
- matplotlib or plotly (chart generation)
- svgwrite (SVG generation)

**Output**:
- Jinja2 (HTML templating)
- Puppeteer (PDF export)
- Markdown (intermediate format)

**Infrastructure**:
- Celery (task queue)
- Redis (cache + queue backend)
- psycopg2 (PostgreSQL driver)
- boto3 (S3 client)

### 4.3 Development Tools

- **Version Control**: Git + GitHub
- **Dependency Management**: Poetry or pip-tools
- **Testing**: pytest, coverage.py
- **Linting**: ruff, black (formatting), mypy (type checking)
- **Documentation**: Sphinx or MkDocs

## 5. Deployment Architecture

### 5.1 Deployment Models

**ARCH-025**: Local Development
- All components run locally
- Docker Compose for service orchestration
- MinIO for S3-compatible storage
- PostgreSQL in container
- Redis in container

**ARCH-026**: Cloud Deployment (AWS)
```
┌──────────────────────────────────────────────────────┐
│  CloudFront (CDN)                                     │
└────────────┬─────────────────────────────────────────┘
             │
┌────────────▼─────────────────────────────────────────┐
│  S3 (Static HTML/PDF outputs)                        │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│  API Gateway → Lambda Functions                      │
│  (Upload, Status, Download endpoints)                │
└────────────┬─────────────────────────────────────────┘
             │
┌────────────▼─────────────────────────────────────────┐
│  ECS/Fargate (Processing Pipeline)                   │
│  - Orchestrator container                            │
│  - Stage worker containers (autoscaling)             │
└────────────┬─────────────────────────────────────────┘
             │
┌────────────▼─────────────────────────────────────────┐
│  RDS PostgreSQL (Metadata)                           │
│  ElastiCache Redis (Queue + Cache)                   │
│  S3 (Raw uploads, guide files, generated images)     │
└──────────────────────────────────────────────────────┘
```

**ARCH-027**: Self-Hosted Deployment
- Docker Compose or Kubernetes
- On-premise PostgreSQL
- Local file storage or NAS
- No external AI API dependency (use local models)

### 5.2 Scalability Considerations

**Horizontal Scaling**:
- Stage workers can scale independently
- Stateless design enables easy parallelization
- Queue-based processing handles load spikes

**Vertical Scaling**:
- Visual generation may require GPU instances
- Large documents may need memory-optimized instances

**Cost Optimization**:
- Cache AI API responses (same input → same output)
- Use cheaper models for simple tasks (classification vs. generation)
- Process visuals asynchronously to avoid blocking

## 6. Security Architecture

**ARCH-028**: Access Control
- User authentication for uploads
- API key authentication for programmatic access
- Role-based access control (admin, user, viewer)

**ARCH-029**: Data Protection
- Encrypt uploads at rest (S3 encryption)
- Encrypt metadata in database (field-level encryption for sensitive data)
- Secure AI API calls (HTTPS, API key rotation)

**ARCH-030**: Content Safety
- Scan uploads for malware
- Sanitize user inputs
- Rate limit API calls
- Audit log for all processing actions

## 7. Monitoring and Observability

**ARCH-031**: Metrics to Track
- **Processing Metrics**: 
  - Time per stage
  - Success/failure rates
  - Retry counts
  - Queue depth
  
- **Quality Metrics**:
  - Validation pass rates
  - Terms defined on first use
  - Visual generation success rate
  
- **System Metrics**:
  - AI API latency and costs
  - Storage usage
  - Database query performance

**ARCH-032**: Logging Strategy
- Structured logging (JSON format)
- Log levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
- Include: stage name, book ID, processing time, guide versions used
- Centralized log aggregation (ELK stack or CloudWatch)

**ARCH-033**: Alerting
- Alert on: stage failures, validation score < threshold, AI API errors
- Notification channels: Email, Slack, PagerDuty
- Escalation policy for critical failures

## 8. Testing Strategy

**ARCH-034**: Unit Testing
- Test each processing stage independently
- Mock AI API responses
- Test guide file parsing
- Coverage target: 80%+

**ARCH-035**: Integration Testing
- Test full pipeline with sample inputs
- Verify metadata flows correctly
- Check guide file updates propagate
- Test failure and retry scenarios

**ARCH-036**: Quality Testing
- Benchmark validation accuracy against human review
- Test with diverse input types (clean vs. messy transcripts)
- Measure output quality with test readers
- A/B test guide file changes

## 9. Future Considerations

**ARCH-037**: Multi-Language Support
- Detect source language
- Translate or process in native language
- Generate glossary in multiple languages

**ARCH-038**: Domain Specialization
- Template guide files for specific domains (tech, business, health)
- Domain-specific terminology databases
- Custom validation rules per domain

**ARCH-039**: Incremental Updates
- Support updating published books with new content
- Track changes and version history
- Regenerate only affected chapters

**ARCH-040**: Interactive Elements
- Add quizzes or exercises
- Embed code playgrounds
- Include video/audio embeds

## 10. Architectural Decision Records (ADRs)

### ADR-001: Why AI-First vs. Human-in-the-Loop?

**Decision**: Build fully automated AI pipeline, not collaborative editorial tool

**Rationale**:
- Target use case is transforming large volumes of content quickly
- Human editorial loops add latency and cost
- AI quality has reached threshold for "good enough" output
- Human review can happen at the end, not during processing

**Consequences**:
- Must invest heavily in validation to catch AI errors
- Need robust guide files to encode editorial knowledge
- Simpler UX (upload → download) but less control

**Alternatives Considered**:
- Human-in-the-loop: Too slow, doesn't scale
- Hybrid (AI drafts, human edits): Requires complex collaboration tools

### ADR-002: Why Vertical Visual Report vs. Book Spread?

**Decision**: Output format is vertical scrolling visual report, not paginated book

**Rationale**:
- Primary consumption is digital (web, tablet, mobile)
- Vertical scrolling is natural for digital reading
- Easier to incorporate rich visuals without page breaks
- Modern expectation for non-fiction content

**Consequences**:
- Cannot directly print as physical book
- Must optimize for screen reading (contrast, font size)
- Responsive design is critical

**Alternatives Considered**:
- Landscape book spread: Simulates physical book but awkward on screens
- EPUB: Good for e-readers but limits visual richness

### ADR-003: Why Guide Files vs. Code Configuration?

**Decision**: Encode all transformation rules in markdown guide files, not Python code

**Rationale**:
- Non-engineers can update guide files
- AI can read guide files directly in prompts
- Versioning is simpler (Git for markdown)
- Changes don't require code deployment

**Consequences**:
- Guide files must be very precise and complete
- AI must reliably follow guide instructions
- Guide file parsing is critical path

**Alternatives Considered**:
- Python config files: Requires engineering for changes
- Database-driven rules: Hard to version and review

### ADR-004: Why PostgreSQL vs. MongoDB?

**Decision**: Use PostgreSQL for metadata storage

**Rationale**:
- Need relational queries (chapters → glossary terms)
- ACID guarantees for validation results
- Full-text search for glossary
- Mature tooling and ecosystem

**Consequences**:
- Schema must be defined upfront
- Less flexible for evolving metadata models
- Need migrations for schema changes

**Alternatives Considered**:
- MongoDB: More flexible but weak relational queries
- SQLite: Too limited for production scale

## 11. Appendix

### 11.1 Glossary of Architectural Terms

- **Guide File**: Markdown document encoding transformation rules, style guidelines, or validation criteria
- **Processing Stage**: Independent component that performs one transformation step
- **Metadata**: Structured data describing content, not the content itself
- **Vertical Visual Report**: Digital document optimized for scrolling, not pagination
- **Validation**: Automated quality check against defined criteria

### 11.2 Related Documents

- Product Brief: High-level overview and value proposition
- PRD: Detailed functional and non-functional requirements
- 13 Guide Files: Operational instructions for AI transformation
- API Documentation: Interface specifications for programmatic access
- Deployment Guide: Instructions for setting up infrastructure

### 11.3 Change History

- **v1.0** (2024-05-05): Initial architecture document
