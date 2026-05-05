# Visual Content Model: Visual Enhancement Guide

## Document Control

**Version**: 1.0  
**Purpose**: Define visual selection, generation, and integration
**Scope**: Visual types, chart rules, image requirements, metadata
**Status**: Active

---

## 1. Visual Selection Rule

**Core Principle**: Use visuals when they reduce cognitive load or reveal structure that text cannot efficiently convey.

**When to add visuals**:
- Complex relationships between components
- Process flows or sequences
- Comparisons across multiple dimensions
- System architecture or hierarchy
- Data patterns or trends
- Before/after transformations
- Spatial arrangements

**When NOT to use visuals**:
- Simple lists (text is clearer)
- Single data points (just state the number)
- Decoration without information
- Redundant with surrounding text

---

## 2. Visual Types

### Type 1: Concept Diagrams

**Purpose**: Illustrate relationships between abstract concepts

**Use for**:
- Showing how concepts relate (hierarchies, dependencies)
- Visualizing mental models
- Mapping connections between ideas

**Example**: Diagram showing relationship between HTTP, REST, and APIs

**Technical specs**:
- SVG format (scalable)
- Clear labels on all elements
- Arrows show directionality
- Color used meaningfully (not decoratively)

### Type 2: Process Flows

**Purpose**: Show sequence of steps or events

**Use for**:
- API request-response cycles
- Authentication flows
- Data transformation pipelines
- Error handling paths

**Example**: Flowchart of OAuth 2.0 authorization flow

**Technical specs**:
- Start/end clearly marked
- Decision points as diamonds
- Process steps as rectangles
- Arrows show flow direction
- Alternate paths visible

### Type 3: Architecture Diagrams

**Purpose**: Show system components and their interactions

**Use for**:
- Client-server architecture
- Microservice layouts
- Database relationships
- Network topologies

**Example**: REST API architecture with client, server, database

**Technical specs**:
- Components as boxes
- Communication lines between components
- Labels on all connections
- Layer separation if applicable

### Type 4: Comparison Tables

**Purpose**: Compare features, approaches, or options

**Use for**:
- REST vs GraphQL vs SOAP
- GET vs POST vs PUT vs DELETE
- Authentication methods comparison

**Example**:

| Feature | GET | POST | PUT | DELETE |
|---------|-----|------|-----|--------|
| Purpose | Retrieve | Create | Update | Remove |
| Idempotent | Yes | No | Yes | Yes |
| Safe | Yes | No | No | No |

**Technical specs**:
- Header row for categories
- Clear row/column alignment
- Maximum 5-6 columns (readability)
- Brief cell content

### Type 5: Timelines

**Purpose**: Show chronological progression

**Use for**:
- History of API standards
- Version evolution
- Project milestones
- Event sequences

**Example**: Timeline of REST API development from 2000 to present

**Technical specs**:
- Horizontal (preferred) or vertical
- Clear date markers
- Events labeled
- Milestones highlighted

### Type 6: Charts/Graphs

**Purpose**: Visualize quantitative data

**Use for**:
- Performance comparisons
- Benchmark results
- Usage trends
- Distribution patterns

**Sub-types**:
- **Bar charts**: Comparing discrete categories
- **Line charts**: Trends over time
- **Pie charts**: Proportions of a whole
- **Scatter plots**: Correlation patterns

**Example**: Bar chart comparing API response times

**Technical specs**:
- Axes labeled with units
- Data series clearly distinguished
- Title describes what's shown
- Legend if multiple series
- Source cited if real data

### Type 7: Annotated Screenshots

**Purpose**: Show actual user interfaces or code

**Use for**:
- API testing tools (Postman, cURL)
- Server responses
- Code editor examples
- Configuration panels

**Example**: Postman screenshot with annotations highlighting request parts

**Technical specs**:
- High resolution (readable text)
- Annotations with arrows/callouts
- Cropped to relevant area
- Updated if UI changes

### Type 8: Callout Boxes

**Purpose**: Highlight key information

**Use for**:
- Important warnings
- Critical tips
- Key definitions
- Examples

**Types**:
- **Info** (blue): Additional context
- **Warning** (yellow/orange): Caution, common mistakes
- **Tip** (green): Best practices, shortcuts
- **Example** (gray): Concrete illustration

**Example**:

```markdown
💡 **Tip**: Always use HTTPS for API requests in production to encrypt data in transit.

⚠️ **Warning**: GET requests with large query parameters may exceed URL length limits.
```

### Type 9: Code Visualizations

**Purpose**: Illustrate code behavior or structure

**Use for**:
- Step-by-step execution
- Variable state changes
- Call stack visualization
- Memory diagrams

**Example**: Diagram showing how a recursive API call works

**Technical specs**:
- Code snippets alongside visual
- State changes highlighted
- Execution order numbered

---

## 3. Chart Rules

### 3.1 Use Real Data When Making Factual Claims

**Bad**: "REST is faster than SOAP" [no chart or generic chart]

**Good**: Chart with real benchmark data + citation: "In a 2023 benchmark by Example Corp testing 10,000 requests..."

**Illustrative data**: If using made-up data for illustration, label clearly:

"**Illustrative Example** (not real benchmark data): The following chart shows how response time might vary..."

### 3.2 Label Everything

**Required labels**:
- Chart title: "API Response Times by Framework"
- X-axis label: "Framework"
- Y-axis label: "Response Time (milliseconds)"
- Data series legend (if multiple series)
- Data point labels (if few data points)

### 3.3 Avoid Decorative Charts

**Bad**: 3D pie chart with gradient fills and drop shadows (looks fancy, harder to read)

**Good**: Flat 2D bar chart with clear colors and direct labels

**Rule**: Every visual element should convey information, not decoration.

### 3.4 Prefer Direct Labels

**Instead of**:
- Legends far from data
- Color coding without labels

**Use**:
- Labels directly on or near data
- Clear text annotations

---

## 4. Image Requirements

### 4.1 Every Image Needs a Caption

**Format**:

```markdown
![Alt text here]

**Figure 3.1**: Caption describing what the image shows and why it matters.
```

**Caption characteristics**:
- Starts with "Figure X.Y" (chapter.number)
- Describes what's shown
- Explains significance (optional)
- 1-3 sentences

**Example**:

```markdown
![Diagram showing HTTP request-response cycle with client sending GET request and server returning JSON]

**Figure 2.1**: The HTTP request-response cycle. The client sends a request, the server processes it, and returns a response. Each request is independent and stateless.
```

### 4.2 Every Visual Must Have Alt Text

**Purpose**: Accessibility for screen readers

**Alt text guidelines**:
- Describe what's shown
- Don't say "image of" or "diagram showing"
- Include key information conveyed by image
- 1-2 sentences

**Example**:

```markdown
![Client computer sending HTTP GET request to server, server processing request and returning JSON response with status 200]
```

### 4.3 Every Visual Must Connect to Nearby Prose

**Pattern**:
1. Introduce visual in text
2. Show visual
3. Explain key insights from visual

**Example**:

```markdown
The request-response cycle involves several steps. Figure 2.1 illustrates this flow:

[IMAGE]

Notice that the client initiates communication and the server responds. This pattern is consistent across all REST API interactions.
```

**Bad (disconnected)**:

```markdown
[IMAGE]

REST APIs use HTTP.
```

---

## 5. Visual Metadata Schema

```yaml
visual:
  # Identification
  id: uuid
  type: enum [diagram, chart, callout, timeline, table, screenshot, code_viz]
  figure_number: string  # e.g., "3.2" (chapter.sequence)
  
  # Content
  title: string
  caption: text
  alt_text: text
  
  # Generation
  generation_method: enum [ai_svg, ai_image_api, manual, extracted]
  source_content_ids: array[string]  # Which text sections this illustrates
  creation_prompt: text  # AI prompt used to generate (if applicable)
  
  # Location
  chapter_id: uuid
  section_title: string
  placement: enum [inline, full_width, margin]
  
  # Technical
  file_path: string
  file_format: enum [svg, png, jpg, webp]
  dimensions:
    width: integer
    height: integer
  file_size_bytes: integer
  
  # Data (if chart)
  data_source: string  # Citation or "illustrative"
  data_points: object  # Structured data used in chart
  
  # Status
  status: enum [planned, generated, reviewed, approved]
  needs_update: boolean
  update_reason: string
  
  # Metadata
  created_date: datetime
  last_updated: datetime
```

---

## 6. Visual Candidate Detection

### 6.1 Text Patterns That Signal Visual Needs

**Process descriptions**:
- "First... then... finally..."
- "The flow involves..."
- "Step 1, Step 2, Step 3..."

→ **Suggest**: Process flow diagram

**Comparisons**:
- "X vs Y"
- "Unlike A, B..."
- "Differences between..."

→ **Suggest**: Comparison table or side-by-side diagram

**System descriptions**:
- "Components include..."
- "The client communicates with the server..."
- "Architecture consists of..."

→ **Suggest**: Architecture diagram

**Data mentions**:
- "Response times averaged 150ms, 200ms, and 180ms..."
- "Usage increased from 1000 to 5000 requests..."
- "Distribution shows..."

→ **Suggest**: Chart (bar, line, pie)

**Relationships**:
- "Depends on..."
- "Related to..."
- "Hierarchy of..."

→ **Suggest**: Concept diagram with connections

---

## 7. Visual Generation Workflow

### 7.1 Planning Phase

1. **Identify candidate**: Text pattern suggests visual would help
2. **Specify visual**:
   - Type (diagram, chart, table, etc.)
   - Content (what to show)
   - Key elements (labels, data, relationships)
   - Purpose (what insight it reveals)

3. **Create specification**:

```yaml
visual_spec:
  type: "process_flow"
  title: "OAuth 2.0 Authorization Flow"
  elements:
    - "User clicks 'Login with Google'"
    - "App redirects to Google"
    - "User authorizes app"
    - "Google returns authorization code"
    - "App exchanges code for token"
    - "App uses token to access data"
  relationships:
    - {from: "User", to: "App", action: "clicks login"}
    - {from: "App", to: "Google", action: "redirects"}
  insights:
    - "User never shares password with app"
    - "Token is obtained in two steps for security"
```

### 7.2 Generation Phase

**Method 1: AI SVG Generation**
- AI generates SVG code directly
- Embed in HTML output
- Scalable and editable

**Method 2: AI Image API**
- Use image generation API (DALL-E, etc.)
- Provide detailed description
- Download and include PNG/JPG

**Method 3: Visualization Library**
- For charts: Use D3.js, Chart.js, Plotly
- Feed data programmatically
- Render to SVG or Canvas

**Method 4: Manual Creation**
- For complex or custom visuals
- Use design tools (Figma, draw.io)
- Export to appropriate format

### 7.3 Integration Phase

1. **Save file**: Store in assets directory
2. **Generate metadata**: Create visual record in database
3. **Insert into chapter**: Add figure number, caption, alt text
4. **Reference in text**: "Figure 3.2 illustrates..."
5. **Validate**: Check that visual enhances understanding

---

## 8. Visual-to-Text Ratio Target

**Goal**: 1 visual per 500-1000 words

**Calculation**:
- 3,000-word chapter → 3-6 visuals
- 500-word section → 0-1 visual

**Quality over quantity**: Better to have 3 great visuals than 10 mediocre ones.

---

## 9. Visual Style Consistency

### 9.1 Design System Elements

**Color palette**:
- Primary: #2563eb (blue)
- Secondary: #10b981 (green)
- Warning: #f59e0b (orange)
- Error: #ef4444 (red)
- Neutral: #6b7280 (gray)

**Typography**:
- Headers: Sans-serif, bold
- Labels: Sans-serif, regular
- Code: Monospace

**Spacing**:
- Margins: 16px standard
- Between elements: 8px minimum

**Shape**:
- Rectangles: Rounded corners (4px radius)
- Lines: 2px width
- Arrows: Clear triangular heads

### 9.2 Accessibility

**Color contrast**: WCAG AA minimum (4.5:1 for text)

**Don't rely on color alone**:
- Use patterns or labels in addition to color
- Example: Success = green + checkmark; Error = red + X

**Text size**: Minimum 12px in diagrams

---

## 10. Visual Validation Checklist

Before finalizing a visual:

✅ **Purpose**:
- [ ] Visual adds information (not decoration)
- [ ] Visual reduces cognitive load vs. text explanation
- [ ] Specific insight is revealed

✅ **Quality**:
- [ ] High enough resolution (readable)
- [ ] Clear labels on all elements
- [ ] Caption describes content and significance
- [ ] Alt text provided for accessibility

✅ **Integration**:
- [ ] Referenced in nearby text
- [ ] Figure number assigned
- [ ] Placed near relevant content
- [ ] Explains key insight after showing visual

✅ **Accuracy**:
- [ ] Data is real (or marked as illustrative)
- [ ] Labels are correct
- [ ] Relationships accurately depicted
- [ ] Citations provided if needed

✅ **Style**:
- [ ] Matches book design system
- [ ] Consistent with other visuals
- [ ] Accessible (color contrast, size)

---

## 11. Visual Updates and Versioning

### 11.1 When to Update Visuals

**Triggers**:
- Source content changed
- Data updated
- UI screenshots outdated (tool version changed)
- Error discovered
- Better approach identified

### 11.2 Update Tracking

```yaml
visual_update:
  visual_id: uuid
  version: integer
  update_reason: string
  changes_made: text
  previous_file: string
  new_file: string
  updated_by: string
  updated_date: datetime
```

---

## 12. Version History

- **v1.0** (2024-05-05): Initial visual content model guide
