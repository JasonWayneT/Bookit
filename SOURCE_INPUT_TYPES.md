# Source Input Types: Classification and Processing Guide

## Document Control

**Version**: 1.0  
**Purpose**: Define all expected source input types and their processing rules  
**Scope**: Classification criteria, extraction methods, transformation rules per type  
**Status**: Active  

---

## 1. Overview

This guide defines nine source input types and provides processing instructions for each. The AI must classify every uploaded file into one of these types to apply the correct transformation rules.

**Input Types**:
1. Transcripts (spoken content → text)
2. Notes (fragmentary ideas)
3. Articles/Essays (coherent prose)
4. Research Dumps (source-heavy material)
5. AI Chats (dialogic material)
6. Slides/Outlines (compressed material)
7. Technical Docs/Code (expert material)
8. Datasets/Tables (structured evidence)
9. Images/Screenshots (visual artifacts)

---

## 2. Type 1: Transcripts

### 2.1 Definition

Transcripts are verbatim recordings of spoken content: lectures, interviews, podcast episodes, meeting recordings, presentations with narration.

### 2.2 Identification Signals

**Strong signals** (classify as transcript if 3+ present):
- Contains verbal filler: "um," "uh," "like," "you know," "sort of"
- Conversational tone: "so," "basically," "right?"
- Speaker attributions: "John:", "[Interviewer]:", timestamps
- Repeated phrases or self-corrections: "what I mean is..."
- Questions and answers in dialogue format
- Timing markers: "[00:15:32]" or SRT format
- Lowercase or inconsistent capitalization (auto-transcription artifact)

**File formats**: .txt, .srt, .vtt, .docx, .pdf

### 2.3 Expected Strengths

- Rich explanatory content (speakers elaborate verbally)
- Natural examples and analogies
- Genuine questions that reveal learning gaps
- Enthusiastic, engaging tone
- Stories and anecdotes

### 2.4 Common Problems

- **Verbal filler**: "um," "uh," "like," "so," "basically"
- **Repetition**: Speakers repeat for emphasis or clarity
- **Tangents**: Off-topic digressions
- **Incomplete sentences**: Started thoughts that trail off
- **Assumed context**: References to shared knowledge ("as we discussed")
- **Unclear references**: "this," "that," "it" without clear antecedents
- **Speaker errors**: Misspoken words, factual errors, retractions
- **Poor punctuation**: Auto-transcription produces run-ons

### 2.5 Processing Rules

**Stage 1: Cleanup**
1. **Remove verbal filler**:
   - Delete: "um," "uh," "like," "you know," "sort of," "kind of," "I mean," "basically"
   - Preserve: Content-carrying instances (e.g., "I mean the word 'algorithm'")

2. **Fix speaker errors**:
   - Remove false starts: "The API—well, actually, the REST API..."
   - Consolidate repetition: "It's important, really important, super important" → "It's very important"
   - Correct self-corrections: "It uses HTTP, or I should say HTTPS" → "It uses HTTPS"

3. **Complete sentences**:
   - Finish trailing thoughts: "If you look at the..." → "If you look at the diagram..."
   - Remove dead-end fragments: "And then we could—never mind."

4. **Add punctuation**:
   - Break run-ons into proper sentences
   - Add commas, periods, question marks as appropriate
   - Capitalize properly

**Stage 2: Structure Detection**
1. **Identify natural sections**:
   - Topic shifts: "Now let's talk about..."
   - Questions: "What is an API?"
   - Speaker transitions: Interviewer questions → expert answers

2. **Extract teaching moments**:
   - Direct explanations: "An API is..."
   - Examples: "For example, imagine..."
   - Analogies: "Think of it like..."
   - Definitions: "We call this..."

3. **Map to book structure**:
   - Major topics → Chapters
   - Subtopics → Sections
   - Examples/anecdotes → Subsections

**Stage 3: Expand and Formalize**
1. **Convert spoken to written**:
   - Spoken: "So basically what happens is the server sends back data"
   - Written: "The server returns data in response to the request"

2. **Make context explicit**:
   - Spoken: "As we discussed earlier..."
   - Written: "In Chapter 2, we explained that REST APIs are stateless..."

3. **Replace vague references**:
   - Spoken: "This thing is important"
   - Written: "Stateless design is important"

4. **Add missing scaffolding**:
   - If speaker assumes knowledge, add definitions
   - If speaker jumps topics, add transitions
   - If speaker references external material, provide context

### 2.6 Metadata Requirements

```yaml
transcript_metadata:
  speaker_count: integer
  duration_estimate: string (e.g., "45 minutes")
  has_timestamps: boolean
  transcript_quality: enum [high, medium, low]
  primary_speakers: array[string]
  topic_keywords: array[string]
  cleanup_applied:
    filler_removed: boolean
    repetition_consolidated: boolean
    punctuation_added: boolean
```

### 2.7 Likely Book Roles

- **Core chapters**: Main explanations from expert speakers
- **Examples**: Anecdotes and stories become concrete examples
- **FAQs**: Questions from audience/interviewer → book sections
- **Glossary**: Terms defined verbally → glossary entries

---

## 3. Type 2: Notes

### 3.1 Definition

Notes are fragmentary ideas, bullet points, outlines, or sketches. They may be personal study notes, brainstorming sessions, meeting minutes, or research notes.

### 3.2 Identification Signals

**Strong signals**:
- Bullet points or numbered lists
- Incomplete sentences or sentence fragments
- Abbreviations and shorthand
- Mixed formatting (headings, sub-bullets, arrows)
- Personal reminders: "TODO:", "Remember:", "Follow up:"
- Non-sequential organization (topics jump around)

**File formats**: .txt, .md, .docx, .pdf, notion exports, Evernote exports

### 3.3 Expected Strengths

- Concise, dense information
- Direct capture of key ideas
- Often well-organized hierarchically
- May include questions or areas needing exploration

### 3.4 Common Problems

- **Incomplete thoughts**: "API design - important"
- **Missing context**: Assumes you know what "the meeting" refers to
- **Duplication**: Same idea noted multiple times in different words
- **No connective tissue**: Just facts, no explanations of relationships
- **Inconsistent depth**: Some items are detailed, others are single words
- **Personal jargon**: Idiosyncratic abbreviations or terms

### 3.5 Processing Rules

**Stage 1: Deduplication and Grouping**
1. **Identify duplicate concepts**:
   - "REST is stateless" and "No session state in REST" → Same concept
   - Consolidate duplicates into single note

2. **Group related items**:
   - All notes about "API security" → One chapter/section
   - Use hierarchical structure in notes as grouping hint

3. **Expand abbreviations**:
   - "DB" → "database"
   - "API" → "Application Programming Interface (API)" on first use
   - Domain-specific abbreviations → Full terms

**Stage 2: Complete Thoughts**
1. **Expand fragments into sentences**:
   - Fragment: "JSON - data format"
   - Sentence: "JSON (JavaScript Object Notation) is a data format commonly used in APIs."

2. **Add connective tissue**:
   - Between items: Add transitions explaining relationships
   - Between groups: Add section introductions

3. **Fill gaps**:
   - If note says "Three types:" but only lists two, flag for research
   - If note references "the example" but none is provided, generate one

**Stage 3: Structure Building**
1. **Convert hierarchy to book structure**:
   - Top-level bullets → Chapters or major sections
   - Sub-bullets → Sections or subsections
   - Nested details → Paragraphs

2. **Determine sequence**:
   - Look for dependency cues: "First," "Then," "After that"
   - Order by prerequisite relationships
   - Use logical progression (simple → complex)

3. **Add examples**:
   - Notes often lack examples—generate concrete examples for abstract points

### 3.6 Metadata Requirements

```yaml
notes_metadata:
  structure_type: enum [hierarchical, flat, mixed]
  abbreviation_density: float
  completeness_score: float (0-1)
  duplication_detected: boolean
  topic_clusters: array[string]
```

### 3.7 Likely Book Roles

- **Chapter outlines**: Hierarchical notes → Chapter structure
- **Key points**: Dense notes → Section content
- **Reference material**: Detailed notes → Appendices or deep-dive sections

---

## 4. Type 3: Articles/Essays

### 4.1 Definition

Articles and essays are coherent, structured prose written for publication: blog posts, magazine articles, white papers, thought pieces, tutorials.

### 4.2 Identification Signals

**Strong signals**:
- Complete sentences and paragraphs
- Clear introduction, body, conclusion structure
- Headings and subheadings
- Polished, edited tone
- Citations or references
- Byline or author attribution

**File formats**: .md, .html, .docx, .pdf

### 4.3 Expected Strengths

- Well-written, coherent prose
- Clear structure and flow
- Good examples and analogies
- Properly cited sources
- Minimal cleanup needed

### 4.4 Common Problems

- **Too broad for a book chapter**: Articles cover large topics superficially
- **Audience mismatch**: Written for experts, not beginners
- **Marketing tone**: Promotional language or product pitches
- **Dated references**: Time-specific examples that no longer apply
- **Assumed context**: References to current events or specific debates

### 4.5 Processing Rules

**Stage 1: Evaluate Fit**
1. **Assess scope**:
   - If article is 5,000+ words → May become entire chapter or multiple sections
   - If article is <2,000 words → May become single section

2. **Check audience level**:
   - If written for experts → Needs beginner scaffolding added
   - If written for general audience → May need technical depth added
   - If written for beginners → May be usable with minimal changes

**Stage 2: Adapt Structure**
1. **Map to book hierarchy**:
   - Article intro → Chapter intro or section intro
   - Article sections → Book sections or subsections
   - Article conclusion → Chapter recap

2. **Adjust transitions**:
   - Article: "In the next section..."
   - Book: "In the next chapter..." or "Later in this chapter..."

3. **Remove article artifacts**:
   - Bylines, publication dates (move to attribution metadata)
   - Call-to-action endings: "Sign up for our newsletter"
   - Promotional content: "Our product solves this"

**Stage 3: Enhance Pedagogy**
1. **Add beginner scaffolding** (if needed):
   - Define technical terms inline
   - Add prerequisite explanations
   - Expand brief mentions into full explanations

2. **Deepen examples** (if needed):
   - Articles often use brief examples—expand them
   - Add more diverse examples if article uses only one

3. **Add visual candidates**:
   - Articles may describe processes without diagrams—flag for visual generation

### 3.6 Metadata Requirements

```yaml
article_metadata:
  original_title: string
  author: string
  publication_date: string
  original_url: string
  word_count: integer
  audience_level: enum [beginner, intermediate, expert]
  completeness: enum [standalone, requires_context]
```

### 4.7 Likely Book Roles

- **Core chapters**: Well-structured articles → Complete chapters
- **Sections**: Shorter articles → Sections within chapters
- **Case studies**: Real-world examples in articles → Extended examples in book

---

## 5. Type 4: Research Dumps

### 5.1 Definition

Research dumps are collections of source material: literature reviews, annotated bibliographies, research notes with citations, compilation of quotes and references.

### 5.2 Identification Signals

**Strong signals**:
- Heavy citation density (references every few sentences)
- Quotes from multiple sources
- Citation format (APA, MLA, footnotes)
- Academic or research tone
- Bibliography or references section
- Multiple perspectives presented

**File formats**: .pdf, .docx, .md, .bib

### 5.3 Expected Strengths

- Well-sourced, evidence-based
- Multiple perspectives and approaches
- Comprehensive coverage of topic
- High factual accuracy (if sources are good)

### 5.4 Common Problems

- **Too dense**: Overwhelming amount of information
- **Lacks synthesis**: Lists sources without integrating ideas
- **Citation overload**: Too many references for a teaching book
- **Contradictory sources**: Different sources make conflicting claims
- **Academic jargon**: Written for researchers, not learners
- **Missing explanations**: Assumes reader understands source context

### 5.5 Processing Rules

**Stage 1: Extract and Synthesize**
1. **Identify main themes**:
   - What are the 3-5 major ideas across all sources?
   - Group sources by theme

2. **Synthesize perspectives**:
   - Don't just list what each source says
   - Integrate: "Three approaches to API design have emerged: [approach 1], [approach 2], [approach 3]"
   - Acknowledge conflicts: "While Smith (2020) argues X, Jones (2021) found evidence for Y"

3. **Simplify citations**:
   - Academic: "According to Johnson et al. (2022), stateless design improves scalability (p. 45)."
   - Book: "Research shows that stateless design improves scalability."
   - Store full citation in metadata, use simplified attribution in text

**Stage 2: Transform to Teaching Prose**
1. **Convert literature review to explanation**:
   - Academic: "Multiple studies have demonstrated... (Smith, 2020; Jones, 2021; Lee, 2022)"
   - Teaching: "Researchers have found that REST APIs scale better than SOAP APIs. This is because..."

2. **Extract key findings**:
   - Don't preserve every detail from every source
   - Identify the most important insights for learners

3. **Add pedagogical structure**:
   - Research dumps often lack teaching sequence
   - Organize by learning progression, not source order

**Stage 3: Verify and Attribute**
1. **Check factual claims**:
   - If sources conflict, note the disagreement or choose the consensus view
   - Flag unverified claims for review

2. **Maintain attribution metadata**:
   - Even if citations are simplified in text, track sources
   - Enable readers to verify claims if needed

3. **Add context for sources**:
   - Don't assume readers know the significance of a study
   - "A 2023 benchmark study by CloudPerf tested..."

### 5.6 Metadata Requirements

```yaml
research_metadata:
  source_count: integer
  citation_format: enum [APA, MLA, Chicago, informal]
  sources:
    - title: string
      author: string
      year: integer
      url: string (optional)
      key_findings: string
  conflicting_claims: array[object]
  synthesis_confidence: float (0-1)
```

### 5.7 Likely Book Roles

- **Evidence-backed chapters**: Research → Authoritative explanations
- **Comparative sections**: Multiple sources → Comparison of approaches
- **References/appendices**: Full citations and further reading

---

## 6. Type 5: AI Chats

### 6.1 Definition

AI chats are dialogic exchanges between a human and an AI assistant (ChatGPT, Claude, etc.) where the human asks questions and the AI provides explanations.

### 6.2 Identification Signals

**Strong signals**:
- Clear user/assistant alternation: "User:", "Assistant:"
- Question-answer format
- AI hedging language: "As an AI," "I cannot," "It's important to note"
- Conversational tone similar to transcripts but more structured
- Code blocks or formatted examples (AI often uses markdown)

**File formats**: .txt, .md, .pdf, screenshots converted to text

### 6.3 Expected Strengths

- Well-explained concepts (AI is designed for clarity)
- Often includes examples and analogies
- Structured responses with lists and headers
- Generally accurate information (with caveats)
- Question-driven organization

### 6.4 Common Problems

- **AI disclaimers**: "As an AI, I don't have personal experience"
- **Hedging**: "Generally," "often," "in many cases" (excessive caution)
- **Repetition**: User asks similar questions, AI gives similar answers
- **Conversational filler**: "Great question!", "I hope this helps!"
- **Inconsistent depth**: Some answers are detailed, others superficial
- **Potential inaccuracies**: AI may hallucinate or provide outdated info
- **Format artifacts**: Markdown formatting, code block syntax

### 6.5 Processing Rules

**Stage 1: Extract Core Content**
1. **Remove conversational filler**:
   - Delete: "Great question!", "I hope this helps!", "Is there anything else?"
   - Delete: AI self-references: "As an AI," "I was trained on"

2. **Consolidate duplicate answers**:
   - If user asks same question multiple ways, combine best answer

3. **Remove hedging** (where appropriate):
   - "This generally works" → "This works"
   - Keep hedging only when genuinely uncertain: "The exact performance depends on..."

**Stage 2: Synthesize Q&A into Prose**
1. **Convert question-answer to explanation**:
   - Q: "What is REST?" A: "REST stands for..."
   - → "REST (Representational State Transfer) is an architectural style for building web services..."

2. **Use questions to structure content**:
   - Questions reveal what learners need to know
   - Organize chapter sections around common questions

3. **Integrate examples**:
   - AI often provides multiple examples across different answers
   - Consolidate best examples

**Stage 3: Verify and Enhance**
1. **Fact-check claims**:
   - AI chats may contain errors or outdated information
   - Cross-reference critical claims

2. **Add missing depth**:
   - AI answers are often introductory—expand as needed
   - If AI says "There are several types," list and explain them all

3. **Remove formatting artifacts**:
   - Code block syntax (```) → Properly formatted code examples
   - Markdown headers → Book-appropriate headers

### 6.6 Metadata Requirements

```yaml
ai_chat_metadata:
  ai_system: string (e.g., "ChatGPT-4", "Claude")
  message_count: integer
  user_question_count: integer
  topics_covered: array[string]
  factual_verification_needed: boolean
```

### 6.7 Likely Book Roles

- **FAQ-style sections**: Questions → Section headings
- **Concept explanations**: AI answers → Expanded explanations
- **Examples**: AI-generated examples → Book examples (verified)

---

## 7. Type 6: Slides/Outlines

### 7.1 Definition

Slides or outlines are compressed, visual-oriented material: PowerPoint slides, Keynote presentations, slide decks, structured outlines.

### 7.2 Identification Signals

**Strong signals**:
- Bullet points with minimal text per point
- Headers without body text: "Introduction" with 3 bullets
- "Slide X" labels or page numbers
- Images or diagram references: "[Diagram: API Flow]"
- Speaker notes sections
- Very brief phrases: "Benefits:", "How it works:"

**File formats**: .pptx, .key, .pdf, .md

### 7.3 Expected Strengths

- Clear structure and hierarchy
- Concise key points
- Visual cues (even if images aren't available)
- Logical flow (designed for presentation)
- Often includes diagrams or charts

### 7.4 Common Problems

- **Extreme compression**: "API: Important" (no explanation)
- **Missing connective tissue**: Bullets without context
- **Assumed narration**: Slides assume speaker will elaborate
- **Jargon without definition**: Assumes audience knows terms
- **Visual-dependent**: "See diagram" but diagram isn't present

### 7.5 Processing Rules

**Stage 1: Decompress and Expand**
1. **Expand bullets into sentences**:
   - Bullet: "REST: Stateless design"
   - Sentence: "REST APIs use a stateless design, meaning each request is independent."

2. **Add explanatory prose between points**:
   - Slides: "Benefits: • Scalability • Simplicity"
   - Book: "REST APIs offer two main benefits. First, scalability: stateless design allows easy horizontal scaling. Second, simplicity: standard HTTP methods make APIs intuitive to use."

3. **Elaborate on headers**:
   - Slide: "Introduction" (header only)
   - Book: Full introductory paragraph explaining the topic

**Stage 2: Integrate Visual References**
1. **Preserve diagram descriptions**:
   - If slide says "See API flow diagram," note this for visual generation

2. **Extract data from charts**:
   - If slide shows bar chart, extract data points for recreation

3. **Create alt text for missing visuals**:
   - Describe what the visual likely showed

**Stage 3: Add Examples**
1. **Slides often lack examples**:
   - Slides: "HTTP Methods: GET, POST, PUT, DELETE"
   - Book: Explain each method with an example

2. **Expand use cases**:
   - Slides: "When to use REST"
   - Book: Detailed scenarios with concrete examples

### 7.6 Metadata Requirements

```yaml
slides_metadata:
  slide_count: integer
  has_speaker_notes: boolean
  visual_references: array[string]
  compression_level: enum [extreme, high, moderate]
  expansion_needed: array[string] (topics needing expansion)
```

### 7.7 Likely Book Roles

- **Chapter structure**: Slide outline → Chapter skeleton
- **Visual plan**: Slide diagrams → Book visual requirements
- **Key points**: Slide bullets → Expanded sections

---

## 8. Type 7: Technical Docs/Code

### 8.1 Definition

Technical documentation or code: API references, SDK documentation, README files, code repositories, technical specifications.

### 8.2 Identification Signals

**Strong signals**:
- Code blocks in multiple languages
- Function/method signatures: `GET /users/:id`
- Parameter lists and return types
- Installation instructions
- Command-line examples: `npm install package`
- Version numbers and compatibility notes

**File formats**: .md, .rst, .txt, code files (.py, .js, .java), .pdf

### 8.3 Expected Strengths

- Precise, accurate technical information
- Complete specifications
- Working code examples
- Clear parameters and return values

### 8.4 Common Problems

- **Expert-only language**: Assumes deep technical knowledge
- **No conceptual explanation**: "Here's how to use it" without "What is it"
- **Missing prerequisites**: Doesn't explain what you need to know first
- **Jargon-heavy**: Every sentence has technical terms
- **Reference-style, not tutorial**: Lists options without teaching when to use them

### 8.5 Processing Rules

**Stage 1: Add Conceptual Foundation**
1. **Start with "What and Why"**:
   - Before showing code, explain the concept
   - Before listing parameters, explain the function's purpose

2. **Add prerequisites**:
   - "To understand this, you first need to know..."
   - Define all technical terms before using them in examples

3. **Provide context**:
   - Don't just show code—explain the problem it solves

**Stage 2: Transform Reference to Tutorial**
1. **Add narrative flow**:
   - Reference: "Parameters: name (string), age (integer)"
   - Tutorial: "The function takes two parameters. First, the user's name as a string..."

2. **Explain options, don't just list them**:
   - Reference: "Options: sync, async, defer"
   - Tutorial: "You have three options for loading scripts. Use 'sync' for critical scripts that must load immediately..."

3. **Show progression**:
   - Start with simplest example
   - Build to more complex examples
   - Explain each addition

**Stage 3: Make Code Accessible**
1. **Add comments to code examples**:
   - Explain what each section does
   - Highlight important lines

2. **Provide multiple examples**:
   - Basic example for beginners
   - Advanced example for deeper understanding

3. **Explain output**:
   - Show what the code returns
   - Explain what each field means

### 8.6 Metadata Requirements

```yaml
technical_docs_metadata:
  doc_type: enum [api_reference, sdk_docs, readme, tutorial, specification]
  language: string (if code-heavy)
  version: string
  beginner_scaffolding_needed: boolean
  code_block_count: integer
```

### 8.7 Likely Book Roles

- **Technical chapters**: Documentation → Beginner-friendly explanations
- **Code examples**: Documentation examples → Annotated, explained examples
- **Reference sections**: API specs → Appendix or quick reference

---

## 9. Type 8: Datasets/Tables

### 9.1 Definition

Datasets or tables: CSV files, spreadsheets, data tables, benchmark results, survey data, statistical summaries.

### 9.2 Identification Signals

**Strong signals**:
- Tabular format (rows and columns)
- Headers with data types: "Name", "Age", "Score"
- Numerical data
- CSV format or Excel files
- Statistical summaries: mean, median, standard deviation

**File formats**: .csv, .xlsx, .tsv, .json (structured data)

### 9.3 Expected Strengths

- Precise, quantitative evidence
- Comparative data (perfect for charts)
- Comprehensive coverage
- Objective information

### 9.4 Common Problems

- **No interpretation**: Just numbers without explanation
- **Missing context**: What do these numbers represent?
- **Unclear units**: Is this milliseconds or seconds?
- **Raw data dump**: Thousands of rows without summary
- **No visual representation**: Tables are hard to interpret without charts

### 9.5 Processing Rules

**Stage 1: Analyze and Summarize**
1. **Extract key insights**:
   - Don't include raw data tables in book
   - Summarize: "The data shows that REST APIs averaged 150ms response time, compared to 200ms for GraphQL."

2. **Identify patterns**:
   - What trends are visible?
   - What comparisons are meaningful?

3. **Provide context**:
   - Where did this data come from?
   - What was the methodology?
   - Why does it matter?

**Stage 2: Create Visualizations**
1. **Convert tables to charts**:
   - Comparison data → Bar chart
   - Time series → Line chart
   - Proportions → Pie chart
   - Distributions → Histogram

2. **Add chart annotations**:
   - Highlight key data points
   - Explain what the chart shows

**Stage 3: Integrate with Text**
1. **Explain before showing**:
   - "We benchmarked five API frameworks. The chart below shows response times:"
   - [Chart]
   - "Notice that REST consistently outperformed SOAP..."

2. **Add interpretation**:
   - What do the numbers mean in practice?
   - What should readers take away?

### 9.6 Metadata Requirements

```yaml
dataset_metadata:
  row_count: integer
  column_count: integer
  data_source: string
  collection_date: string
  units: object (map of column -> unit)
  summary_statistics: object
```

### 9.7 Likely Book Roles

- **Evidence**: Data → Charts with interpretation
- **Comparisons**: Tables → Comparative analysis
- **Benchmarks**: Performance data → Performance section

---

## 10. Type 9: Images/Screenshots

### 10.1 Definition

Images or screenshots: diagrams, photos, UI screenshots, charts, infographics, handwritten notes (scanned).

### 10.2 Identification Signals

**Strong signals**:
- Image file formats: .png, .jpg, .gif, .svg
- Visual content without text extraction
- Screenshots of UIs, code editors, terminals
- Diagrams or flowcharts
- Scanned documents

**File formats**: .png, .jpg, .jpeg, .gif, .svg, .pdf (image-based)

### 10.3 Expected Strengths

- Visual representation already exists
- May show real-world examples (screenshots)
- Diagrams may illustrate complex concepts
- Infographics may be well-designed

### 10.4 Common Problems

- **No explanation**: Image without caption or context
- **Low quality**: Blurry or hard to read
- **Missing alt text**: Not accessible
- **Outdated**: UI screenshots from old versions
- **Requires OCR**: Handwritten notes need text extraction
- **Not editable**: Can't update or modify diagram

### 10.5 Processing Rules

**Stage 1: Extract Information**
1. **OCR if needed**:
   - Scanned handwritten notes → Text extraction
   - Screenshots with text → Text extraction for reference

2. **Describe visual**:
   - What does the image show?
   - What concepts does it illustrate?

3. **Extract data** (if applicable):
   - Charts → Data points
   - Diagrams → Process steps or relationships

**Stage 2: Integrate with Book**
1. **Add caption**:
   - Every image needs a descriptive caption
   - "Figure 3.1: REST API request-response cycle"

2. **Add alt text**:
   - Accessibility requirement
   - "Diagram showing client sending HTTP request to server, server processing request, and server returning JSON response to client"

3. **Reference in text**:
   - "The diagram below illustrates the request-response cycle."
   - [Image]
   - "Notice how each request is independent..."

**Stage 3: Consider Recreation**
1. **Evaluate quality**:
   - If image is low quality, flag for recreation
   - If image is outdated, flag for update

2. **Extract for consistency**:
   - If image uses different style than book, consider recreating
   - Maintain consistent visual style across book

### 10.6 Metadata Requirements

```yaml
image_metadata:
  original_filename: string
  format: string
  dimensions: {width: integer, height: integer}
  file_size: integer
  description: string
  alt_text: string
  contains_text: boolean
  quality_score: float (0-1)
  recreation_needed: boolean
```

### 10.7 Likely Book Roles

- **Visuals**: Images → Book figures with captions
- **UI examples**: Screenshots → Annotated examples
- **Concept illustration**: Diagrams → Teaching visuals

---

## 11. Classification Decision Tree

When classifying uploaded content, use this decision tree:

```
1. Is it primarily spoken content recorded as text?
   YES → TRANSCRIPT
   NO → Continue

2. Is it fragmentary/bullet points with incomplete sentences?
   YES → Are they organized as presentation slides?
          YES → SLIDES/OUTLINES
          NO → NOTES
   NO → Continue

3. Is it coherent prose with introduction, body, conclusion?
   YES → Does it have heavy citation density?
          YES → RESEARCH DUMP
          NO → ARTICLE/ESSAY
   NO → Continue

4. Is it question-answer dialogue format?
   YES → Are answers from an AI assistant?
          YES → AI CHAT
          NO → Classify as TRANSCRIPT or NOTES
   NO → Continue

5. Is it technical documentation with code examples?
   YES → TECHNICAL DOCS/CODE
   NO → Continue

6. Is it tabular/structured data?
   YES → DATASETS/TABLES
   NO → Continue

7. Is it an image file?
   YES → IMAGES/SCREENSHOTS
   NO → Flag as UNKNOWN, default to NOTES
```

---

## 12. Multi-Type Sources

Some sources may combine multiple types:
- **Transcript with slides**: Process as transcript, use slides for structure hints
- **Article with embedded data**: Process as article, extract datasets separately
- **Technical docs with images**: Process docs, integrate images

**Rule**: Classify by **primary** content type, note secondary types in metadata.

---

## 13. Quality Scoring

For each source, assess quality:

```yaml
quality_assessment:
  clarity: float (0-1) # How clear is the content?
  completeness: float (0-1) # Does it cover the topic fully?
  accuracy_confidence: float (0-1) # How confident are we it's correct?
  transformation_difficulty: enum [easy, moderate, hard]
  scaffolding_needed: enum [minimal, moderate, extensive]
```

Use quality scores to prioritize processing effort and set validation thresholds.

---

## 14. Version History

- **v1.0** (2024-05-05): Initial source input types guide
