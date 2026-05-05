# Book System: Technical Design Document

## Document Control

**Version**: 1.0  
**Date**: 2024-05-05  
**Status**: Design  
**Related Documents**:
- Product Brief: High-level overview
- PRD: Requirements (FR-001 to FR-030, NFR-001 to NFR-011)
- Architecture: System architecture and ADRs
- 13 Guide Files: Operational rules for AI transformation

---

## 1. Design Overview

### 1.1 Purpose

This document provides implementation-ready technical specifications for the Book System. It translates requirements from the PRD into concrete technical designs that engineering can implement.

### 1.2 Design Principles

**DP-1**: **Guide-File Driven Architecture**
- All transformation logic is encoded in versioned markdown guide files
- AI reads guide files in prompts (not hardcoded rules)
- Changes to behavior happen via guide file updates, not code changes

**DP-2**: **Stateless Pipeline Stages**
- Each stage (Intake, Structure, Expansion, etc.) is independently executable
- State persisted in database between stages
- Failed stages can retry without affecting other stages

**DP-3**: **Metadata-First**
- Every decision, transformation, and artifact is tracked in metadata
- Enables debugging, auditing, and quality analysis
- Supports reproducibility and experimentation

**DP-4**: **Progressive Enhancement**
- System produces valid output at each stage
- Early stages create basic books, later stages add sophistication
- Visual generation failure doesn't block publication

**DP-5**: **AI-Native Validation**
- AI validates its own output against guide files
- Automated quality checks before human review
- Self-healing through retry with refined prompts

---

## 2. Component Design

### 2.1 Intake Engine

**Satisfies**: FR-001, FR-002, FR-003, FR-004, FR-005, FR-006

#### 2.1.1 Component Interface

```python
class IntakeEngine:
    """
    Handles file upload, text extraction, content classification,
    and source metadata creation.
    """
    
    def process_upload(
        self,
        file_path: str,
        uploaded_by: str,
        metadata: Optional[Dict] = None
    ) -> SourceItem:
        """
        Process uploaded file through intake pipeline.
        
        Args:
            file_path: Path to uploaded file
            uploaded_by: User ID or system identifier
            metadata: Optional user-provided metadata
            
        Returns:
            SourceItem with classification and extracted content
            
        Raises:
            FileCorruptedError: File cannot be read
            UnsupportedFormatError: File format not supported
            ExtractionError: Text extraction failed
        """
        pass
    
    def extract_text(self, file_path: str, file_type: str) -> ExtractedContent:
        """Extract text from file based on type."""
        pass
    
    def classify_content(
        self,
        text: str,
        file_format: str
    ) -> ContentClassification:
        """Classify content type using AI and heuristics."""
        pass
    
    def create_source_metadata(
        self,
        file_info: FileInfo,
        extracted: ExtractedContent,
        classification: ContentClassification
    ) -> SourceItem:
        """Generate complete source item metadata."""
        pass
```

#### 2.1.2 Text Extraction Logic

**Design Decision**: Use format-specific libraries, fallback to generic extraction

**Implementation**:

```python
# text_extractor.py

class TextExtractor:
    def extract(self, file_path: str, file_type: str) -> ExtractedContent:
        """Route to appropriate extractor based on file type."""
        
        extractors = {
            'txt': self._extract_plain_text,
            'md': self._extract_plain_text,
            'pdf': self._extract_pdf,
            'docx': self._extract_docx,
            'srt': self._extract_srt,
            'vtt': self._extract_vtt,
        }
        
        extractor = extractors.get(file_type, self._extract_generic)
        
        try:
            return extractor(file_path)
        except Exception as e:
            logger.error(f"Extraction failed: {e}")
            raise ExtractionError(f"Could not extract text from {file_type}")
    
    def _extract_pdf(self, file_path: str) -> ExtractedContent:
        """Extract text from PDF using pdfplumber."""
        import pdfplumber
        
        text_blocks = []
        metadata = {'page_count': 0}
        
        with pdfplumber.open(file_path) as pdf:
            metadata['page_count'] = len(pdf.pages)
            
            for page_num, page in enumerate(pdf.pages, 1):
                text = page.extract_text()
                if text:
                    text_blocks.append({
                        'page': page_num,
                        'text': text
                    })
        
        full_text = '\n\n'.join(block['text'] for block in text_blocks)
        
        return ExtractedContent(
            text=full_text,
            structure=text_blocks,
            metadata=metadata,
            quality=self._assess_quality(full_text)
        )
    
    def _extract_docx(self, file_path: str) -> ExtractedContent:
        """Extract text from DOCX preserving structure."""
        from docx import Document
        
        doc = Document(file_path)
        
        # Extract with structure preservation
        structured_content = []
        for para in doc.paragraphs:
            structured_content.append({
                'type': 'paragraph',
                'style': para.style.name,
                'text': para.text
            })
        
        # Extract tables if present
        for table in doc.tables:
            structured_content.append({
                'type': 'table',
                'rows': [[cell.text for cell in row.cells] for row in table.rows]
            })
        
        full_text = '\n\n'.join(
            item['text'] if item['type'] == 'paragraph' else str(item)
            for item in structured_content
        )
        
        return ExtractedContent(
            text=full_text,
            structure=structured_content,
            metadata={'paragraph_count': len(doc.paragraphs)},
            quality=self._assess_quality(full_text)
        )
    
    def _extract_srt(self, file_path: str) -> ExtractedContent:
        """Extract text from SRT subtitle file."""
        import srt
        
        with open(file_path, 'r', encoding='utf-8') as f:
            subtitle_generator = srt.parse(f.read())
            subtitles = list(subtitle_generator)
        
        # Extract structured content with timestamps
        structured = []
        for sub in subtitles:
            structured.append({
                'start': sub.start.total_seconds(),
                'end': sub.end.total_seconds(),
                'text': sub.content
            })
        
        # Full text without timestamps
        full_text = '\n'.join(sub.content for sub in subtitles)
        
        return ExtractedContent(
            text=full_text,
            structure=structured,
            metadata={'subtitle_count': len(subtitles)},
            quality=self._assess_quality(full_text)
        )
    
    def _assess_quality(self, text: str) -> str:
        """Assess extraction quality based on text characteristics."""
        
        # Heuristics for quality assessment
        has_sentences = '.' in text
        has_words = len(text.split()) > 10
        has_proper_encoding = not any(c in text for c in ['\ufffd', '�'])
        
        if has_sentences and has_words and has_proper_encoding:
            return 'high'
        elif has_words:
            return 'medium'
        else:
            return 'low'
```

#### 2.1.3 Content Classification

**Design Decision**: Hybrid approach (AI + heuristics) for robustness

**Implementation**:

```python
# content_classifier.py

class ContentClassifier:
    """Classify content type using AI with heuristic fallback."""
    
    def __init__(self, ai_client, guide_file_loader):
        self.ai_client = ai_client
        self.guide_loader = guide_file_loader
    
    def classify(
        self,
        text: str,
        file_format: str,
        filename: str
    ) -> ContentClassification:
        """
        Classify content type.
        
        Returns:
            ContentClassification with type and confidence
        """
        
        # Step 1: Quick heuristic check
        heuristic_result = self._heuristic_classification(text, file_format)
        
        if heuristic_result.confidence >= 0.9:
            # High confidence heuristic, skip AI
            return heuristic_result
        
        # Step 2: AI classification
        ai_result = self._ai_classification(text, file_format, filename)
        
        # Step 3: Combine results
        if ai_result.confidence >= 0.7:
            return ai_result
        elif heuristic_result.confidence >= 0.5:
            return heuristic_result
        else:
            # Default to 'notes' with low confidence
            return ContentClassification(
                type='notes',
                confidence=0.3,
                method='default'
            )
    
    def _heuristic_classification(
        self,
        text: str,
        file_format: str
    ) -> ContentClassification:
        """Fast heuristic-based classification."""
        
        # Load heuristics from guide file
        guide = self.guide_loader.load('SOURCE_INPUT_TYPES.md')
        
        signals = {
            'transcript': self._detect_transcript_signals(text),
            'notes': self._detect_notes_signals(text),
            'article': self._detect_article_signals(text),
            'slides': self._detect_slides_signals(text),
        }
        
        # Count matching signals
        max_signals = max(signals.values())
        content_type = max(signals, key=signals.get)
        
        # Confidence based on signal strength
        confidence = min(max_signals / 5.0, 0.95)  # 5+ signals = high confidence
        
        return ContentClassification(
            type=content_type,
            confidence=confidence,
            method='heuristic',
            signals=signals
        )
    
    def _detect_transcript_signals(self, text: str) -> int:
        """Count transcript signals in text."""
        signals = 0
        
        # Verbal filler
        filler_words = ['um', 'uh', 'like', 'you know', 'sort of']
        for filler in filler_words:
            if re.search(rf'\b{filler}\b', text, re.IGNORECASE):
                signals += 1
                break
        
        # Speaker attributions
        if re.search(r'^[A-Z][a-z]+:', text, re.MULTILINE):
            signals += 1
        
        # Timestamps
        if re.search(r'\[\d{2}:\d{2}:\d{2}\]', text):
            signals += 1
        
        # Conversational tone
        if re.search(r'\b(so|basically|right\?|you know what I mean)\b', text, re.IGNORECASE):
            signals += 1
        
        # Self-corrections
        if re.search(r'(—well,|I mean,|actually)', text, re.IGNORECASE):
            signals += 1
        
        return signals
    
    def _ai_classification(
        self,
        text: str,
        file_format: str,
        filename: str
    ) -> ContentClassification:
        """AI-based classification using Claude."""
        
        # Load classification rules from guide
        guide_content = self.guide_loader.load('SOURCE_INPUT_TYPES.md')
        
        # Sample text for classification (first 2000 words)
        sample = ' '.join(text.split()[:2000])
        
        prompt = f"""Read the source input types guide below, then classify the provided content.

{guide_content}

Now classify this content:

Filename: {filename}
Format: {file_format}
Content sample:
{sample}

Respond with JSON:
{{
    "type": "transcript|notes|article|research|ai_chat|slides|technical_doc|dataset|image",
    "confidence": 0.0-1.0,
    "reasoning": "brief explanation"
}}"""
        
        response = self.ai_client.generate(
            prompt=prompt,
            temperature=0.1,  # Low temperature for consistent classification
            max_tokens=200
        )
        
        # Parse JSON response
        result = json.loads(response.strip())
        
        return ContentClassification(
            type=result['type'],
            confidence=result['confidence'],
            method='ai',
            reasoning=result.get('reasoning', '')
        )
```

#### 2.1.4 Metadata Generation

**Design Decision**: Comprehensive metadata from the start enables downstream traceability

**Implementation**:

```python
# metadata_generator.py

class MetadataGenerator:
    """Generate source item metadata."""
    
    def create_source_item(
        self,
        file_info: FileInfo,
        extracted: ExtractedContent,
        classification: ContentClassification
    ) -> SourceItem:
        """Create complete source item with metadata."""
        
        # Generate unique ID
        source_id = self._generate_id()
        
        # Compute file hash for deduplication
        file_hash = self._compute_hash(file_info.path)
        
        # Extract topic keywords
        keywords = self._extract_keywords(extracted.text)
        
        # Detect technical terms
        detected_terms = self._detect_terms(extracted.text)
        
        # Assess quality
        quality_score = self._assess_quality(extracted, classification)
        
        return SourceItem(
            # Identification
            id=source_id,
            filename=file_info.filename,
            file_hash=file_hash,
            
            # Classification
            content_type=classification.type,
            classification_confidence=classification.confidence,
            classification_method=classification.method,
            
            # Content
            raw_text=extracted.text,
            file_size_bytes=file_info.size,
            word_count=len(extracted.text.split()),
            encoding=file_info.encoding,
            language='en-US',  # TODO: Add language detection
            
            # Extraction metadata
            extraction_method=extracted.method,
            extraction_quality=extracted.quality,
            extraction_issues=extracted.issues,
            
            # Processing status
            status='classified',
            processing_started=datetime.utcnow(),
            processing_completed=None,
            retry_count=0,
            
            # Source tracking
            original_url=None,
            upload_date=datetime.utcnow(),
            uploaded_by=file_info.uploaded_by,
            
            # Content analysis
            topic_keywords=keywords,
            detected_terms=detected_terms,
            quality_score=quality_score,
            
            # Relationships
            derived_books=[],
            combined_with_sources=[],
            
            # Metadata
            created_date=datetime.utcnow(),
            last_modified=datetime.utcnow(),
            notes=None
        )
    
    def _extract_keywords(self, text: str, top_n: int = 10) -> List[str]:
        """Extract topic keywords using TF-IDF."""
        from sklearn.feature_extraction.text import TfidfVectorizer
        
        # Simple keyword extraction
        vectorizer = TfidfVectorizer(
            max_features=top_n,
            stop_words='english',
            ngram_range=(1, 2)
        )
        
        try:
            tfidf_matrix = vectorizer.fit_transform([text])
            keywords = vectorizer.get_feature_names_out()
            return list(keywords)
        except:
            # Fallback: most common capitalized words
            words = re.findall(r'\b[A-Z][a-z]+\b', text)
            return list(set(words))[:top_n]
    
    def _detect_terms(self, text: str) -> List[str]:
        """Detect technical terms (capitalized words, acronyms)."""
        
        # Acronyms (all caps, 2+ letters)
        acronyms = set(re.findall(r'\b[A-Z]{2,}\b', text))
        
        # Technical terms (capitalized words appearing multiple times)
        capitalized = re.findall(r'\b[A-Z][a-z]+\b', text)
        term_counts = {}
        for term in capitalized:
            term_counts[term] = term_counts.get(term, 0) + 1
        
        # Terms appearing 3+ times
        frequent_terms = {term for term, count in term_counts.items() if count >= 3}
        
        all_terms = acronyms | frequent_terms
        return sorted(all_terms)
```

---

### 2.2 Structure Engine

**Satisfies**: FR-007, FR-008, FR-009, FR-010, FR-011

#### 2.2.1 Component Interface

```python
class StructureEngine:
    """
    Analyzes content and builds book hierarchy.
    Detects prerequisites, identifies gaps, creates structure.
    """
    
    def generate_structure(
        self,
        source_item: SourceItem
    ) -> BookStructure:
        """
        Generate book hierarchy from source content.
        
        Returns:
            BookStructure with parts, chapters, sections, gaps
        """
        pass
    
    def detect_prerequisites(
        self,
        chapters: List[ChapterOutline]
    ) -> PrerequisiteGraph:
        """Build prerequisite dependency graph."""
        pass
    
    def identify_gaps(
        self,
        content: str,
        structure: BookStructure
    ) -> List[ContentGap]:
        """Detect missing explanations or assumed knowledge."""
        pass
    
    def sequence_chapters(
        self,
        chapters: List[ChapterOutline],
        prerequisites: PrerequisiteGraph
    ) -> List[ChapterOutline]:
        """Order chapters based on prerequisites."""
        pass
```

#### 2.2.2 Hierarchy Generation Algorithm

**Design Decision**: AI-driven structure generation with validation

**Algorithm**:

```python
# structure_generator.py

class StructureGenerator:
    """Generate book structure using AI."""
    
    def __init__(self, ai_client, guide_loader, validator):
        self.ai_client = ai_client
        self.guide_loader = guide_loader
        self.validator = validator
    
    def generate(self, source_item: SourceItem) -> BookStructure:
        """Generate complete book structure."""
        
        # Step 1: Load guide files
        architecture_guide = self.guide_loader.load('BOOK_ARCHITECTURE.md')
        pedagogy_guide = self.guide_loader.load('PEDAGOGY_RULES.md')
        
        # Step 2: Analyze content for major themes
        themes = self._extract_themes(source_item.raw_text)
        
        # Step 3: Generate chapter outlines
        chapters = self._generate_chapters(
            source_item.raw_text,
            themes,
            architecture_guide,
            pedagogy_guide
        )
        
        # Step 4: Detect prerequisites
        prereq_graph = self._detect_prerequisites(chapters)
        
        # Step 5: Sequence chapters
        ordered_chapters = self._sequence_chapters(chapters, prereq_graph)
        
        # Step 6: Identify content gaps
        gaps = self._identify_gaps(source_item.raw_text, ordered_chapters)
        
        # Step 7: Determine if parts are needed
        uses_parts = len(ordered_chapters) >= 8  # >50 pages estimated
        
        parts = []
        if uses_parts:
            parts = self._create_parts(ordered_chapters)
        
        # Step 8: Create book structure
        structure = BookStructure(
            title=self._generate_title(themes),
            uses_parts=uses_parts,
            parts=parts,
            chapters=ordered_chapters,
            prerequisite_graph=prereq_graph,
            detected_gaps=gaps,
            progression_pattern=self._determine_progression(ordered_chapters)
        )
        
        # Step 9: Validate structure
        validation = self.validator.validate_structure(structure)
        
        if not validation.passed:
            # Regenerate with constraints
            structure = self._regenerate_with_fixes(structure, validation.issues)
        
        return structure
    
    def _generate_chapters(
        self,
        text: str,
        themes: List[str],
        architecture_guide: str,
        pedagogy_guide: str
    ) -> List[ChapterOutline]:
        """Use AI to generate chapter outlines."""
        
        # Sample long text (first 20k words for structure analysis)
        text_sample = ' '.join(text.split()[:20000])
        
        prompt = f"""You are generating a book structure. Read the guides below, then create chapter outlines.

BOOK ARCHITECTURE GUIDE:
{architecture_guide}

PEDAGOGY RULES GUIDE:
{pedagogy_guide}

CONTENT THEMES IDENTIFIED:
{json.dumps(themes, indent=2)}

CONTENT SAMPLE:
{text_sample}

Generate a structured outline with 5-12 chapters. Each chapter must have:
1. ONE primary teaching objective (specific and testable)
2. List of prerequisite concepts
3. 3-6 supporting sections
4. List of terms to introduce
5. Example ideas

Return JSON array of chapters:
[
  {{
    "chapter_number": 1,
    "title": "Chapter Title",
    "teaching_objective": "Specific objective",
    "prerequisite_concepts": ["concept1", "concept2"],
    "sections": [
      {{"title": "Section Title", "supporting_concept": "one concept"}}
    ],
    "terms_to_introduce": ["term1", "term2"],
    "example_ideas": ["example description"]
  }}
]"""
        
        response = self.ai_client.generate(
            prompt=prompt,
            temperature=0.3,
            max_tokens=4000
        )
        
        # Parse JSON response
        chapters_data = json.loads(response.strip())
        
        # Convert to ChapterOutline objects
        chapters = []
        for data in chapters_data:
            chapters.append(ChapterOutline(**data))
        
        return chapters
    
    def _detect_prerequisites(
        self,
        chapters: List[ChapterOutline]
    ) -> PrerequisiteGraph:
        """Build dependency graph between chapters."""
        
        graph = PrerequisiteGraph()
        
        # Add all chapters as nodes
        for chapter in chapters:
            graph.add_node(chapter.chapter_number, chapter.title)
        
        # Analyze prerequisite relationships
        for chapter in chapters:
            for prereq_concept in chapter.prerequisite_concepts:
                # Find which earlier chapter introduces this concept
                for earlier_chapter in chapters:
                    if earlier_chapter.chapter_number >= chapter.chapter_number:
                        continue
                    
                    # Check if concept is in terms_to_introduce
                    if prereq_concept.lower() in [
                        term.lower() for term in earlier_chapter.terms_to_introduce
                    ]:
                        graph.add_edge(
                            earlier_chapter.chapter_number,
                            chapter.chapter_number,
                            prereq_concept
                        )
        
        # Detect circular dependencies
        if graph.has_cycle():
            raise StructureError("Circular prerequisite dependency detected")
        
        return graph
    
    def _sequence_chapters(
        self,
        chapters: List[ChapterOutline],
        prereq_graph: PrerequisiteGraph
    ) -> List[ChapterOutline]:
        """Order chapters using topological sort."""
        
        # Topological sort to respect prerequisites
        sorted_ids = prereq_graph.topological_sort()
        
        # Reorder chapters
        chapter_map = {ch.chapter_number: ch for ch in chapters}
        ordered = [chapter_map[ch_id] for ch_id in sorted_ids]
        
        # Renumber chapters
        for i, chapter in enumerate(ordered, 1):
            chapter.chapter_number = i
        
        return ordered
```

---

### 2.3 Expansion Engine

**Satisfies**: FR-012, FR-013, FR-014, FR-015, FR-016, FR-017

#### 2.3.1 Component Interface

```python
class ExpansionEngine:
    """
    Transforms raw content into beginner-accessible teaching prose.
    Adds definitions, scaffolding, examples, and pedagogical enhancements.
    """
    
    def expand_chapter(
        self,
        chapter_outline: ChapterOutline,
        source_content: str,
        glossary: Glossary
    ) -> ExpandedChapter:
        """
        Expand chapter outline into full teaching content.
        
        Returns:
            ExpandedChapter with complete prose, examples, glossary updates
        """
        pass
    
    def detect_terms(self, text: str) -> List[str]:
        """Identify technical terms requiring definition."""
        pass
    
    def generate_definitions(
        self,
        terms: List[str],
        context: str
    ) -> Dict[str, TermDefinition]:
        """Generate plain-English and technical definitions."""
        pass
    
    def add_scaffolding(
        self,
        text: str,
        complexity_signals: List[str]
    ) -> str:
        """Insert beginner scaffolding for complex concepts."""
        pass
    
    def generate_examples(
        self,
        concept: str,
        context: str
    ) -> List[Example]:
        """Create concrete examples for abstract concepts."""
        pass
```

#### 2.3.2 Chapter Expansion Algorithm

**Design Decision**: Template-driven expansion with AI filling sections

**Implementation**:

```python
# chapter_expander.py

class ChapterExpander:
    """Expand chapter outlines into complete teaching content."""
    
    def __init__(self, ai_client, guide_loader, glossary_manager):
        self.ai_client = ai_client
        self.guide_loader = guide_loader
        self.glossary = glossary_manager
    
    def expand(
        self,
        chapter_outline: ChapterOutline,
        source_content: str
    ) -> ExpandedChapter:
        """Expand chapter using template + AI generation."""
        
        # Load guide files
        template = self.guide_loader.load('CHAPTER_TEMPLATE.md')
        style_guide = self.guide_loader.load('BOOK_STYLE_MANIFEST.md')
        pedagogy_guide = self.guide_loader.load('PEDAGOGY_RULES.md')
        vocab_guide = self.guide_loader.load('VOCABULARY_AND_GLOSSARY_MODEL.md')
        
        # Extract relevant source content for this chapter
        relevant_content = self._extract_relevant_content(
            source_content,
            chapter_outline
        )
        
        # Generate chapter opening
        opening = self._generate_opening(
            chapter_outline,
            relevant_content,
            template,
            style_guide
        )
        
        # Expand each section
        expanded_sections = []
        for section_outline in chapter_outline.sections:
            section = self._expand_section(
                section_outline,
                relevant_content,
                pedagogy_guide,
                vocab_guide
            )
            expanded_sections.append(section)
        
        # Generate recap
        recap = self._generate_recap(
            chapter_outline,
            expanded_sections,
            template
        )
        
        # Generate bridge to next chapter
        bridge = self._generate_bridge(
            chapter_outline,
            template
        )
        
        # Detect and define all terms
        all_text = opening + '\n\n' + '\n\n'.join(
            s.content for s in expanded_sections
        )
        terms_used = self._detect_terms(all_text)
        term_definitions = self._generate_definitions(
            terms_used,
            all_text,
            vocab_guide
        )
        
        # Update glossary
        for term, definition in term_definitions.items():
            self.glossary.add_term(
                term=term,
                definition=definition,
                chapter_id=chapter_outline.id
            )
        
        # Assemble complete chapter
        expanded = ExpandedChapter(
            id=chapter_outline.id,
            chapter_number=chapter_outline.chapter_number,
            title=chapter_outline.title,
            teaching_objective=chapter_outline.teaching_objective,
            opening=opening,
            sections=expanded_sections,
            recap=recap,
            bridge_to_next=bridge,
            terms_introduced=list(term_definitions.keys()),
            word_count=len(all_text.split()),
            status='expanded'
        )
        
        return expanded
    
    def _expand_section(
        self,
        section_outline: SectionOutline,
        source_content: str,
        pedagogy_guide: str,
        vocab_guide: str
    ) -> ExpandedSection:
        """Expand a single section with full teaching content."""
        
        prompt = f"""Expand this section into beginner-friendly teaching content.

PEDAGOGY GUIDE:
{pedagogy_guide}

VOCABULARY GUIDE:
{vocab_guide}

SECTION TO EXPAND:
Title: {section_outline.title}
Supporting Concept: {section_outline.supporting_concept}

SOURCE CONTENT:
{source_content}

Follow this structure:
1. Concept introduction (1-2 paragraphs): What is this concept?
2. Explanation (2-4 paragraphs): How does it work?
3. Example (1-3 paragraphs or code): Concrete illustration
4. Application (1 paragraph): When/why to use this

REQUIREMENTS:
- Define all technical terms on first use
- Use plain English (beginner-friendly)
- Include realistic example
- One concept per paragraph
- No jargon stacking
- Length: 300-800 words

Generate the section content:"""
        
        section_content = self.ai_client.generate(
            prompt=prompt,
            temperature=0.4,
            max_tokens=2000
        )
        
        return ExpandedSection(
            title=section_outline.title,
            supporting_concept=section_outline.supporting_concept,
            content=section_content,
            word_count=len(section_content.split())
        )
```

#### 2.3.3 Term Detection and Definition

**Implementation**:

```python
# term_detector.py

class TermDetector:
    """Detect and define technical terms."""
    
    def detect(self, text: str) -> List[str]:
        """Identify terms requiring definition."""
        
        terms = set()
        
        # 1. Acronyms (ALL CAPS, 2+ letters)
        acronyms = re.findall(r'\b[A-Z]{2,}\b', text)
        terms.update(acronyms)
        
        # 2. Capitalized technical terms (repeated 2+ times)
        capitalized = re.findall(r'\b[A-Z][a-z]+\b', text)
        term_counts = {}
        for term in capitalized:
            if term in ['The', 'A', 'An', 'I']:  # Skip common words
                continue
            term_counts[term] = term_counts.get(term, 0) + 1
        
        frequent = {term for term, count in term_counts.items() if count >= 2}
        terms.update(frequent)
        
        # 3. Compound terms (e.g., "REST API", "HTTP method")
        compound_pattern = r'\b[A-Z]+\s+[A-Z][a-z]+\b'
        compounds = re.findall(compound_pattern, text)
        terms.update(compounds)
        
        # 4. Domain-specific patterns
        # e.g., words ending in common tech suffixes
        tech_suffixes = ['API', 'protocol', 'service', 'endpoint', 'method']
        for suffix in tech_suffixes:
            pattern = rf'\b\w+{suffix}\b'
            matches = re.findall(pattern, text, re.IGNORECASE)
            terms.update(matches)
        
        return sorted(terms)

class TermDefiner:
    """Generate term definitions."""
    
    def __init__(self, ai_client, guide_loader):
        self.ai_client = ai_client
        self.guide_loader = guide_loader
    
    def define(
        self,
        term: str,
        context: str
    ) -> TermDefinition:
        """Generate plain-English and technical definitions."""
        
        vocab_guide = self.guide_loader.load('VOCABULARY_AND_GLOSSARY_MODEL.md')
        
        # Extract context sentences containing the term
        sentences = re.split(r'[.!?]', context)
        context_sentences = [s for s in sentences if term in s][:3]
        context_text = '. '.join(context_sentences)
        
        prompt = f"""Define this term following the vocabulary guide.

VOCABULARY GUIDE:
{vocab_guide}

TERM: {term}

CONTEXT:
{context_text}

Provide:
1. Plain-English definition (1-2 sentences, no jargon)
2. Technical definition (precise, may use domain terms)
3. Example (concrete illustration)
4. Non-example (optional, if helps clarify)

Return JSON:
{{
  "term": "{term}",
  "plain_definition": "...",
  "technical_definition": "...",
  "example": "...",
  "non_example": "..." (or null)
}}"""
        
        response = self.ai_client.generate(
            prompt=prompt,
            temperature=0.2,
            max_tokens=500
        )
        
        data = json.loads(response.strip())
        
        return TermDefinition(
            term=data['term'],
            plain_definition=data['plain_definition'],
            technical_definition=data.get('technical_definition'),
            example=data.get('example'),
            non_example=data.get('non_example')
        )
```

---

### 2.4 Visual Engine

**Satisfies**: FR-018, FR-019, FR-020, FR-021, FR-022

#### 2.4.1 Visual Candidate Detection

**Implementation**:

```python
# visual_detector.py

class VisualCandidateDetector:
    """Identify where visuals would enhance understanding."""
    
    def detect_candidates(
        self,
        chapter_text: str,
        chapter_structure: ChapterOutline
    ) -> List[VisualCandidate]:
        """Scan text for visual opportunities."""
        
        candidates = []
        
        # Load visual guide
        visual_guide = self.guide_loader.load('VISUAL_CONTENT_MODEL.md')
        
        # Pattern matching for common visual triggers
        patterns = {
            'process_flow': [
                r'first\s+\.\.\.\s+then\s+\.\.\.\s+finally',
                r'step\s+\d+',
                r'the\s+flow\s+involves',
                r'sequence\s+of\s+events'
            ],
            'comparison': [
                r'\bvs\b',
                r'unlike\s+\w+,\s+\w+',
                r'difference\s+between',
                r'compared\s+to'
            ],
            'architecture': [
                r'components?\s+include',
                r'client\s+\w+\s+server',
                r'architecture\s+consists',
                r'system\s+design'
            ],
            'data_chart': [
                r'\d+\s*ms\b',
                r'response\s+time',
                r'performance\s+\w+\s+\d+',
                r'benchmark'
            ]
        }
        
        # Scan text with patterns
        for visual_type, pattern_list in patterns.items():
            for pattern in pattern_list:
                matches = re.finditer(pattern, chapter_text, re.IGNORECASE)
                for match in matches:
                    # Extract surrounding context
                    start = max(0, match.start() - 200)
                    end = min(len(chapter_text), match.end() + 200)
                    context = chapter_text[start:end]
                    
                    candidates.append(VisualCandidate(
                        type=visual_type,
                        trigger_text=match.group(),
                        context=context,
                        position=match.start(),
                        confidence=0.7
                    ))
        
        # Deduplicate nearby candidates
        candidates = self._deduplicate_candidates(candidates)
        
        return candidates
    
    def _deduplicate_candidates(
        self,
        candidates: List[VisualCandidate]
    ) -> List[VisualCandidate]:
        """Remove overlapping or nearby candidates."""
        
        if not candidates:
            return []
        
        # Sort by position
        sorted_candidates = sorted(candidates, key=lambda c: c.position)
        
        # Keep candidates at least 500 chars apart
        deduped = [sorted_candidates[0]]
        for candidate in sorted_candidates[1:]:
            if candidate.position - deduped[-1].position > 500:
                deduped.append(candidate)
        
        return deduped
```

#### 2.4.2 Diagram Generation

**Design Decision**: Use AI to generate SVG directly for diagrams

**Implementation**:

```python
# diagram_generator.py

class DiagramGenerator:
    """Generate diagrams from text descriptions."""
    
    def __init__(self, ai_client, guide_loader):
        self.ai_client = ai_client
        self.guide_loader = guide_loader
    
    def generate(
        self,
        candidate: VisualCandidate,
        chapter_context: str
    ) -> GeneratedVisual:
        """Generate diagram based on visual candidate."""
        
        visual_guide = self.guide_loader.load('VISUAL_CONTENT_MODEL.md')
        
        # Determine diagram type
        diagram_type = self._determine_diagram_type(candidate)
        
        # Generate diagram
        if diagram_type == 'process_flow':
            svg_code = self._generate_flowchart(candidate, visual_guide)
        elif diagram_type == 'architecture':
            svg_code = self._generate_architecture_diagram(candidate, visual_guide)
        elif diagram_type == 'comparison':
            svg_code = self._generate_comparison_table(candidate, visual_guide)
        else:
            svg_code = self._generate_generic_diagram(candidate, visual_guide)
        
        # Generate caption and alt text
        caption = self._generate_caption(candidate, chapter_context)
        alt_text = self._generate_alt_text(svg_code, candidate)
        
        return GeneratedVisual(
            type=diagram_type,
            svg_code=svg_code,
            caption=caption,
            alt_text=alt_text,
            file_format='svg',
            generation_method='ai_svg'
        )
    
    def _generate_flowchart(
        self,
        candidate: VisualCandidate,
        visual_guide: str
    ) -> str:
        """Generate SVG flowchart."""
        
        prompt = f"""Generate an SVG flowchart based on this description.

VISUAL GUIDE (for styling):
{visual_guide}

PROCESS DESCRIPTION:
{candidate.context}

Generate clean SVG code with:
- viewBox="0 0 700 400" for responsive scaling
- Use CSS variables for colors (--color-primary, --color-secondary)
- Clear labels on all shapes
- Arrows showing flow direction
- Rectangle shapes for process steps
- Diamond shapes for decisions

Return ONLY the SVG code, no explanation:"""
        
        svg_code = self.ai_client.generate(
            prompt=prompt,
            temperature=0.3,
            max_tokens=2000
        )
        
        # Extract SVG (remove any markdown fences)
        svg_code = re.sub(r'```svg\n|```\n|```', '', svg_code).strip()
        
        # Validate SVG
        if not svg_code.startswith('<svg'):
            raise VisualGenerationError("Invalid SVG generated")
        
        return svg_code
```

---

### 2.5 Validation Engine

**Satisfies**: FR-023, FR-024, FR-025, FR-026, NFR-005

#### 2.5.1 Validation Orchestrator

**Implementation**:

```python
# validation_orchestrator.py

class ValidationOrchestrator:
    """Runs all validation checklists and aggregates results."""
    
    def __init__(self, checklist_loader, validators):
        self.checklist_loader = checklist_loader
        self.validators = validators
    
    def validate_book(self, book: Book) -> ValidationReport:
        """Run all validation checklists on complete book."""
        
        # Load all checklists
        checklists = self.checklist_loader.load('VALIDATION_CHECKLISTS.md')
        
        results = {}
        
        # Run each category of validation
        results['pedagogical'] = self.validators.pedagogical.validate(book)
        results['style'] = self.validators.style.validate(book)
        results['completeness'] = self.validators.completeness.validate(book)
        results['attribution'] = self.validators.attribution.validate(book)
        
        # Calculate overall score
        weights = {
            'pedagogical': 0.30,
            'style': 0.25,
            'completeness': 0.25,
            'attribution': 0.20
        }
        
        overall_score = sum(
            results[category].score * weights[category]
            for category in weights
        )
        
        # Check pass threshold
        passed = overall_score >= 0.8 and all(
            not result.has_critical_failures
            for result in results.values()
        )
        
        # Compile issues
        all_issues = []
        for category, result in results.items():
            all_issues.extend(result.issues)
        
        return ValidationReport(
            overall_score=overall_score,
            passed=passed,
            category_results=results,
            total_checks=sum(r.checks_run for r in results.values()),
            checks_passed=sum(r.checks_passed for r in results.values()),
            checks_failed=sum(r.checks_failed for r in results.values()),
            critical_failures=sum(r.critical_failures for r in results.values()),
            issues=all_issues,
            timestamp=datetime.utcnow()
        )
```

#### 2.5.2 Pedagogical Validator

**Implementation**:

```python
# pedagogical_validator.py

class PedagogicalValidator:
    """Validates pedagogical quality."""
    
    def validate(self, book: Book) -> ValidationCategoryResult:
        """Run pedagogical checklist."""
        
        checks = []
        
        # Check: All terms defined on first use
        term_check = self._check_terms_defined(book)
        checks.append(term_check)
        
        # Check: Examples follow explanations
        example_check = self._check_examples_placement(book)
        checks.append(example_check)
        
        # Check: Scaffolding for complex concepts
        scaffolding_check = self._check_scaffolding(book)
        checks.append(scaffolding_check)
        
        # Check: Prerequisites explained
        prereq_check = self._check_prerequisites(book)
        checks.append(prereq_check)
        
        # Calculate score
        passed_count = sum(1 for c in checks if c.passed)
        score = passed_count / len(checks)
        
        # Identify critical failures
        critical_failures = sum(
            1 for c in checks if c.critical and not c.passed
        )
        
        # Compile issues
        issues = [
            c.to_issue() for c in checks if not c.passed
        ]
        
        return ValidationCategoryResult(
            category='pedagogical',
            score=score,
            checks_run=len(checks),
            checks_passed=passed_count,
            checks_failed=len(checks) - passed_count,
            critical_failures=critical_failures,
            checks=checks,
            issues=issues
        )
    
    def _check_terms_defined(self, book: Book) -> CheckResult:
        """Verify all technical terms defined on first use."""
        
        # Get all terms from glossary
        glossary_terms = set(book.glossary.terms.keys())
        
        # Scan chapters for term usage
        undefined_uses = []
        
        for chapter in book.chapters:
            # Get terms used in this chapter
            terms_in_chapter = self._extract_terms(chapter.content)
            
            # Check each term
            for term in terms_in_chapter:
                if term in glossary_terms:
                    # Check if defined before use in this chapter
                    term_def = book.glossary.terms[term]
                    
                    if term_def.first_appearance_chapter > chapter.chapter_number:
                        undefined_uses.append({
                            'term': term,
                            'chapter': chapter.chapter_number,
                            'defined_in': term_def.first_appearance_chapter
                        })
        
        # Calculate percentage
        total_terms = len(glossary_terms)
        defined_properly = total_terms - len(undefined_uses)
        percentage = defined_properly / total_terms if total_terms > 0 else 1.0
        
        passed = percentage >= 0.9  # 90% threshold
        
        return CheckResult(
            name='terms_defined_on_first_use',
            category='pedagogical',
            passed=passed,
            critical=True,
            score=percentage,
            details=f"{percentage*100:.1f}% of terms defined properly ({defined_properly}/{total_terms})",
            failures=undefined_uses,
            recommendation="Define terms before or at first use"
        )
```

---

### 2.6 Output Engine

**Satisfies**: FR-027, FR-028, FR-029, FR-030, NFR-008

#### 2.6.1 HTML Renderer

**Implementation**:

```python
# html_renderer.py

class HTMLRenderer:
    """Render book as responsive HTML."""
    
    def __init__(self, design_system, template_engine):
        self.design_system = design_system
        self.template_engine = template_engine
    
    def render(self, book: Book) -> HTMLOutput:
        """Render complete book as HTML."""
        
        # Load HTML template
        template = self.template_engine.load('book_template.html')
        
        # Apply design system (CSS variables, typography, spacing)
        css = self.design_system.generate_css()
        
        # Render table of contents
        toc_html = self._render_toc(book)
        
        # Render chapters
        chapters_html = []
        for chapter in book.chapters:
            chapter_html = self._render_chapter(chapter, book)
            chapters_html.append(chapter_html)
        
        # Render glossary
        glossary_html = self._render_glossary(book.glossary)
        
        # Assemble complete HTML
        html = template.render(
            title=book.title,
            subtitle=book.subtitle,
            css=css,
            toc=toc_html,
            chapters=chapters_html,
            glossary=glossary_html,
            metadata=book.metadata
        )
        
        # Minify HTML (optional)
        if self.config.minify:
            html = self._minify_html(html)
        
        return HTMLOutput(
            html=html,
            file_size=len(html.encode('utf-8')),
            render_time=time.time() - start_time
        )
    
    def _render_chapter(self, chapter: Chapter, book: Book) -> str:
        """Render single chapter with all elements."""
        
        chapter_template = self.template_engine.load('chapter.html')
        
        # Render sections
        sections_html = []
        for section in chapter.sections:
            section_html = self._render_section(section)
            sections_html.append(section_html)
        
        # Render visuals
        visuals_html = {}
        for visual in chapter.visuals:
            visual_html = self._render_visual(visual)
            visuals_html[visual.id] = visual_html
        
        # Process content to insert visuals at correct positions
        content_with_visuals = self._insert_visuals(
            chapter.content,
            visuals_html
        )
        
        return chapter_template.render(
            chapter_number=chapter.chapter_number,
            title=chapter.title,
            teaching_objective=chapter.teaching_objective,
            opening=chapter.opening,
            sections=sections_html,
            recap=chapter.recap,
            bridge=chapter.bridge_to_next
        )
```

---

## 3. Data Models (Implementation)

### 3.1 Database Schema

**Design Decision**: PostgreSQL for relational data, JSONB for flexible metadata

**Schema**:

```sql
-- Source Items
CREATE TABLE source_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    filename VARCHAR(255) NOT NULL,
    file_hash VARCHAR(64) UNIQUE,
    content_type VARCHAR(50),
    classification_confidence DECIMAL(3,2),
    raw_text TEXT,
    file_size_bytes BIGINT,
    word_count INTEGER,
    encoding VARCHAR(20),
    extraction_quality VARCHAR(20),
    status VARCHAR(50),
    topic_keywords JSONB,
    detected_terms JSONB,
    quality_score DECIMAL(3,2),
    created_date TIMESTAMP DEFAULT NOW(),
    last_modified TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_source_content_type ON source_items(content_type);
CREATE INDEX idx_source_status ON source_items(status);

-- Books
CREATE TABLE books (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    subtitle VARCHAR(255),
    version VARCHAR(20),
    audience_level VARCHAR(50),
    uses_parts BOOLEAN DEFAULT FALSE,
    chapter_count INTEGER,
    word_count INTEGER,
    validation_score DECIMAL(3,2),
    status VARCHAR(50),
    current_stage VARCHAR(50),
    guide_files_used JSONB,
    published_date TIMESTAMP,
    created_date TIMESTAMP DEFAULT NOW(),
    last_modified TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_book_status ON books(status);

-- Book-Source Relationships
CREATE TABLE book_sources (
    book_id UUID REFERENCES books(id) ON DELETE CASCADE,
    source_id UUID REFERENCES source_items(id) ON DELETE CASCADE,
    is_primary BOOLEAN DEFAULT FALSE,
    PRIMARY KEY (book_id, source_id)
);

-- Chapters
CREATE TABLE chapters (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    book_id UUID REFERENCES books(id) ON DELETE CASCADE,
    part_id UUID,
    chapter_number INTEGER NOT NULL,
    title VARCHAR(255) NOT NULL,
    teaching_objective TEXT,
    opening TEXT,
    core_content TEXT,
    chapter_recap TEXT,
    bridge_to_next TEXT,
    word_count INTEGER,
    status VARCHAR(50),
    validation_results JSONB,
    created_date TIMESTAMP DEFAULT NOW(),
    last_modified TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_chapter_book ON chapters(book_id);
CREATE INDEX idx_chapter_number ON chapters(book_id, chapter_number);

-- Glossary Terms
CREATE TABLE glossary_terms (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    book_id UUID REFERENCES books(id) ON DELETE CASCADE,
    term VARCHAR(255) NOT NULL,
    term_lowercase VARCHAR(255),
    aliases JSONB,
    plain_definition TEXT,
    technical_definition TEXT,
    example TEXT,
    first_appearance_chapter_id UUID REFERENCES chapters(id),
    usage_count INTEGER DEFAULT 0,
    status VARCHAR(50),
    created_date TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_term_book ON glossary_terms(book_id);
CREATE INDEX idx_term_lowercase ON glossary_terms(term_lowercase);
CREATE UNIQUE INDEX idx_term_unique ON glossary_terms(book_id, term_lowercase);

-- Visuals
CREATE TABLE visuals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    book_id UUID REFERENCES books(id) ON DELETE CASCADE,
    chapter_id UUID REFERENCES chapters(id) ON DELETE CASCADE,
    figure_number VARCHAR(20),
    type VARCHAR(50),
    title VARCHAR(255),
    caption TEXT,
    alt_text TEXT,
    file_path VARCHAR(500),
    file_format VARCHAR(10),
    generation_method VARCHAR(50),
    dimensions JSONB,
    status VARCHAR(50),
    created_date TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_visual_chapter ON visuals(chapter_id);

-- Validation Records
CREATE TABLE validation_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    book_id UUID REFERENCES books(id) ON DELETE CASCADE,
    chapter_id UUID,
    validation_timestamp TIMESTAMP DEFAULT NOW(),
    overall_score DECIMAL(3,2),
    passed BOOLEAN,
    category_scores JSONB,
    checks_run INTEGER,
    checks_passed INTEGER,
    issues JSONB,
    requires_revision BOOLEAN
);

CREATE INDEX idx_validation_book ON validation_records(book_id);
CREATE INDEX idx_validation_timestamp ON validation_records(validation_timestamp DESC);

-- Processing Jobs (for async pipeline)
CREATE TABLE processing_jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_type VARCHAR(50) NOT NULL,
    book_id UUID REFERENCES books(id) ON DELETE CASCADE,
    status VARCHAR(50) DEFAULT 'queued',
    progress_percentage DECIMAL(5,2) DEFAULT 0,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    duration_seconds INTEGER,
    retry_count INTEGER DEFAULT 0,
    output JSONB,
    errors JSONB,
    created_date TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_job_status ON processing_jobs(status);
CREATE INDEX idx_job_book ON processing_jobs(book_id);
```

---

## 4. API Design

### 4.1 REST API Endpoints

**Design Decision**: RESTful API for external integration

**Endpoints**:

```yaml
# Upload Source Content
POST /api/v1/sources
Request:
  Content-Type: multipart/form-data
  Body:
    file: <uploaded file>
    metadata: {
      "title": "optional title",
      "notes": "optional notes"
    }
Response:
  Status: 201 Created
  Body:
    {
      "source_id": "uuid",
      "filename": "string",
      "status": "uploaded",
      "content_type": null,
      "classification_confidence": null
    }

# Get Source Status
GET /api/v1/sources/{source_id}
Response:
  Status: 200 OK
  Body:
    {
      "source_id": "uuid",
      "filename": "string",
      "status": "classified",
      "content_type": "transcript",
      "classification_confidence": 0.95,
      "word_count": 7500,
      "created_date": "ISO timestamp"
    }

# Generate Book from Source
POST /api/v1/books
Request:
  Body:
    {
      "source_ids": ["uuid1", "uuid2"],
      "options": {
        "audience_level": "beginner",
        "target_length": "medium"
      }
    }
Response:
  Status: 202 Accepted
  Body:
    {
      "book_id": "uuid",
      "status": "processing",
      "current_stage": "intake",
      "estimated_completion": "ISO timestamp"
    }

# Get Book Status
GET /api/v1/books/{book_id}
Response:
  Status: 200 OK
  Body:
    {
      "book_id": "uuid",
      "title": "string",
      "status": "validated",
      "current_stage": "output",
      "progress_percentage": 85.0,
      "validation_score": 0.87,
      "chapter_count": 8,
      "word_count": 25000,
      "estimated_pages": 125
    }

# Get Book Content
GET /api/v1/books/{book_id}/content
Query Parameters:
  format: html|pdf
Response:
  Status: 200 OK
  Content-Type: text/html or application/pdf
  Body: <rendered book>

# Get Validation Report
GET /api/v1/books/{book_id}/validation
Response:
  Status: 200 OK
  Body:
    {
      "overall_score": 0.87,
      "passed": true,
      "category_results": {
        "pedagogical": {"score": 0.90, "checks_passed": 18, "checks_failed": 2},
        "style": {"score": 0.85, "checks_passed": 17, "checks_failed": 3}
      },
      "issues": [
        {
          "type": "missing_example",
          "severity": "warning",
          "chapter": 3,
          "description": "Chapter 3 has only 1 example"
        }
      ]
    }

# Retry Failed Stage
POST /api/v1/books/{book_id}/retry
Request:
  Body:
    {
      "stage": "expansion",
      "chapter_ids": ["uuid1"] (optional, for chapter-specific retry)
    }
Response:
  Status: 202 Accepted
  Body:
    {
      "job_id": "uuid",
      "status": "queued"
    }
```

---

## 5. Processing Pipeline Implementation

### 5.1 Pipeline Orchestrator

**Design Decision**: Celery task queue for async, distributed processing

**Implementation**:

```python
# pipeline_orchestrator.py

from celery import Celery, chain

app = Celery('book_system', broker='redis://localhost:6379/0')

class PipelineOrchestrator:
    """Orchestrates the 6-stage book generation pipeline."""
    
    def process_book(
        self,
        source_ids: List[str],
        options: Dict
    ) -> str:
        """
        Launch async pipeline for book generation.
        
        Returns:
            book_id for tracking
        """
        
        # Create book record
        book = self._create_book_record(source_ids, options)
        
        # Build pipeline chain
        pipeline = chain(
            intake_task.s(book.id, source_ids),
            structure_task.s(book.id),
            expansion_task.s(book.id),
            visual_task.s(book.id),
            validation_task.s(book.id),
            output_task.s(book.id)
        )
        
        # Launch pipeline
        result = pipeline.apply_async()
        
        # Store job ID
        self.db.update_book(
            book.id,
            {'job_id': result.id, 'status': 'processing'}
        )
        
        return book.id

# Celery Tasks

@app.task(bind=True, max_retries=3)
def intake_task(self, book_id: str, source_ids: List[str]):
    """Stage 1: Intake and classification."""
    try:
        engine = IntakeEngine()
        
        # Process each source
        for source_id in source_ids:
            source = db.get_source(source_id)
            processed = engine.process_upload(source.file_path, source.uploaded_by)
            db.update_source(source_id, processed)
        
        # Update book status
        db.update_book(book_id, {
            'status': 'intake_complete',
            'current_stage': 'structure'
        })
        
        return {'book_id': book_id, 'stage': 'intake', 'status': 'complete'}
        
    except Exception as exc:
        # Retry with exponential backoff
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)

@app.task(bind=True, max_retries=3)
def structure_task(self, previous_result: Dict, book_id: str):
    """Stage 2: Structure generation."""
    try:
        engine = StructureEngine()
        
        # Get source content
        sources = db.get_book_sources(book_id)
        combined_content = '\n\n'.join(s.raw_text for s in sources)
        
        # Generate structure
        structure = engine.generate_structure(combined_content)
        
        # Save structure
        db.save_book_structure(book_id, structure)
        
        # Update status
        db.update_book(book_id, {
            'status': 'structure_complete',
            'current_stage': 'expansion',
            'chapter_count': len(structure.chapters)
        })
        
        return {'book_id': book_id, 'stage': 'structure', 'status': 'complete'}
        
    except Exception as exc:
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)

@app.task(bind=True, max_retries=3)
def expansion_task(self, previous_result: Dict, book_id: str):
    """Stage 3: Content expansion."""
    try:
        engine = ExpansionEngine()
        
        # Get book structure and sources
        book = db.get_book(book_id)
        sources = db.get_book_sources(book_id)
        combined_content = '\n\n'.join(s.raw_text for s in sources)
        
        # Expand each chapter
        for chapter_outline in book.structure.chapters:
            expanded = engine.expand_chapter(
                chapter_outline,
                combined_content,
                book.glossary
            )
            db.save_chapter(book_id, expanded)
        
        # Update status
        db.update_book(book_id, {
            'status': 'expansion_complete',
            'current_stage': 'visual',
            'word_count': sum(c.word_count for c in book.chapters)
        })
        
        return {'book_id': book_id, 'stage': 'expansion', 'status': 'complete'}
        
    except Exception as exc:
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)

# Similar tasks for visual_task, validation_task, output_task...
```

---

## 6. Guide File Integration

### 6.1 Guide File Loader

**Design Decision**: Cache guide files in memory, reload on version change

**Implementation**:

```python
# guide_file_loader.py

class GuideFileLoader:
    """Load and cache guide files."""
    
    def __init__(self, guide_file_path: str):
        self.guide_path = guide_file_path
        self.cache = {}
        self.versions = {}
    
    def load(self, guide_name: str) -> str:
        """
        Load guide file content.
        
        Args:
            guide_name: e.g., 'CONTENT_GUIDE.md'
            
        Returns:
            Full content of guide file
        """
        
        file_path = os.path.join(self.guide_path, guide_name)
        
        # Check if file exists
        if not os.path.exists(file_path):
            raise GuideFileNotFoundError(f"Guide file not found: {guide_name}")
        
        # Check modification time
        mtime = os.path.getmtime(file_path)
        
        # Return cached version if unchanged
        if guide_name in self.cache and self.versions.get(guide_name) == mtime:
            return self.cache[guide_name]
        
        # Load and cache
        with open(file_path, 'r', encoding='utf-8') as f:
            content = f.read()
        
        self.cache[guide_name] = content
        self.versions[guide_name] = mtime
        
        logger.info(f"Loaded guide file: {guide_name} (version: {mtime})")
        
        return content
    
    def get_version(self, guide_name: str) -> str:
        """Get version identifier for guide file."""
        file_path = os.path.join(self.guide_path, guide_name)
        mtime = os.path.getmtime(file_path)
        return f"v{int(mtime)}"
```

---

## 7. Error Handling and Recovery

### 7.1 Error Classification

**Design Decision**: Recoverable vs unrecoverable errors with different handling

**Implementation**:

```python
# error_handler.py

class ErrorHandler:
    """Centralized error handling."""
    
    RECOVERABLE_ERRORS = [
        'ExtractionError',
        'ClassificationUncertain',
        'AIAPITimeout',
        'ValidationFailure'
    ]
    
    UNRECOVERABLE_ERRORS = [
        'FileCorruptedError',
        'UnsupportedFormatError',
        'CircularDependencyError'
    ]
    
    def handle(self, error: Exception, context: Dict) -> ErrorResolution:
        """
        Handle error and determine resolution.
        
        Returns:
            ErrorResolution with action (retry, skip, fail, manual)
        """
        
        error_type = type(error).__name__
        
        # Unrecoverable - fail immediately
        if error_type in self.UNRECOVERABLE_ERRORS:
            return ErrorResolution(
                action='fail',
                message=str(error),
                requires_manual=True
            )
        
        # Recoverable - attempt retry
        if error_type in self.RECOVERABLE_ERRORS:
            retry_count = context.get('retry_count', 0)
            
            if retry_count < 3:
                return ErrorResolution(
                    action='retry',
                    message=f"Retrying (attempt {retry_count + 1}/3)",
                    requires_manual=False,
                    retry_delay=2 ** retry_count  # Exponential backoff
                )
            else:
                return ErrorResolution(
                    action='manual',
                    message="Max retries exceeded, flagging for manual review",
                    requires_manual=True
                )
        
        # Unknown error - log and fail safe
        logger.error(f"Unknown error type: {error_type}", exc_info=True)
        return ErrorResolution(
            action='fail',
            message="Unknown error, failing safely",
            requires_manual=True
        )
```

---

## 8. Testing Strategy

### 8.1 Unit Tests

**Coverage**: All component interfaces

```python
# test_intake_engine.py

import pytest
from book_system.intake import IntakeEngine, TextExtractor

class TestTextExtractor:
    
    def test_extract_plain_text(self, sample_txt_file):
        """Test plain text extraction."""
        extractor = TextExtractor()
        result = extractor.extract(sample_txt_file, 'txt')
        
        assert result.text is not None
        assert len(result.text) > 0
        assert result.quality in ['high', 'medium', 'low']
    
    def test_extract_pdf(self, sample_pdf_file):
        """Test PDF text extraction."""
        extractor = TextExtractor()
        result = extractor.extract(sample_pdf_file, 'pdf')
        
        assert result.text is not None
        assert result.metadata['page_count'] > 0
    
    def test_extract_docx(self, sample_docx_file):
        """Test DOCX extraction with structure."""
        extractor = TextExtractor()
        result = extractor.extract(sample_docx_file, 'docx')
        
        assert result.text is not None
        assert result.structure is not None
        assert len(result.structure) > 0

class TestContentClassifier:
    
    def test_heuristic_transcript_detection(self):
        """Test transcript detection via heuristics."""
        classifier = ContentClassifier(mock_ai_client, mock_guide_loader)
        
        transcript_text = """
        So, um, REST APIs are really important, you know?
        Speaker 1: Yeah, I agree.
        Speaker 2: Basically, they use HTTP methods.
        """
        
        result = classifier._heuristic_classification(transcript_text, 'txt')
        
        assert result.type == 'transcript'
        assert result.confidence > 0.5
```

### 8.2 Integration Tests

```python
# test_pipeline_integration.py

class TestPipelineIntegration:
    
    def test_full_pipeline_simple_transcript(self, test_transcript):
        """Test complete pipeline with simple transcript."""
        
        # Upload source
        source_id = api_client.upload_source(test_transcript)
        
        # Generate book
        book_id = api_client.generate_book([source_id])
        
        # Wait for completion
        book = wait_for_completion(book_id, timeout=300)
        
        # Assertions
        assert book.status == 'validated'
        assert book.chapter_count >= 3
        assert book.validation_score >= 0.8
        assert book.word_count > 2000
    
    def test_pipeline_with_validation_failure(self, low_quality_source):
        """Test pipeline handles validation failures."""
        
        source_id = api_client.upload_source(low_quality_source)
        book_id = api_client.generate_book([source_id])
        
        book = wait_for_completion(book_id, timeout=300)
        
        # Should complete but flag issues
        assert book.status in ['validated', 'needs_revision']
        
        validation = api_client.get_validation(book_id)
        assert len(validation.issues) > 0
```

---

## 9. Deployment Architecture

### 9.1 Container Strategy

**Design Decision**: Docker containers with docker-compose for local, Kubernetes for production

**docker-compose.yml**:

```yaml
version: '3.8'

services:
  web:
    build: ./web
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/book_system
      - REDIS_URL=redis://redis:6379/0
      - AI_API_KEY=${CLAUDE_API_KEY}
    depends_on:
      - db
      - redis
    volumes:
      - ./guide-files:/app/guide-files:ro
  
  worker:
    build: ./worker
    command: celery -A tasks worker --loglevel=info
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/book_system
      - REDIS_URL=redis://redis:6379/0
      - AI_API_KEY=${CLAUDE_API_KEY}
    depends_on:
      - db
      - redis
    volumes:
      - ./guide-files:/app/guide-files:ro
  
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=book_system
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres-data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres-data:
```

---

## 10. Version History

- **v1.0** (2024-05-05): Initial design document
