# Paste to Markdown

A tool to convert rich text or web articles into Markdown format.

## Purpose

Two input modes:
1. **Rich text**: Copy formatted text from a website, paste here, get Markdown
2. **URL**: Paste an article URL, tool fetches and extracts the main content as Markdown

## Output Modes

### Copy Mode (default)
- Output: Markdown text, auto-selected for copying

### File Mode
- Output: bash/fish command to create a file
- User provides an "area" prefix (e.g., "reading")
- Filename: `{area}.{slugified-title}.md`

Example output:
```bash
cat > "reading.how-to-build-a-web-app.md" << 'EOF'
title: How to Build a Web App
author: John Doe
date: 2026-01-18
source: https://example.com

Content here...
EOF
```

## UI Components

### Header Bar
- Mode toggle: Copy / File
- Area input (visible in file mode)

### State 1: Empty (Initial)
- Full-page paste target
- Instructions text: "Paste content or URL (Ctrl+V)"

### State 2: Loading (URL only)
- Loading indicator while fetching and parsing URL

### State 3: Result
- Contenteditable div displaying the output with syntax highlighting
- All text automatically selected for immediate copy (Ctrl+C)
- Press Escape to reset to State 1

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| turndown | latest | Convert HTML to Markdown |
| @mozilla/readability | latest | Extract article content from web pages |
| spellcaster | latest | Reactive state management |
| prismjs | latest | Markdown syntax highlighting |

## Key Functions

### `handlePaste(event)`
- Intercepts paste event
- Detects if content is a URL or rich text
- Routes to appropriate handler

### `isUrl(text)`
- Returns true if text matches URL pattern (http/https)

### `fetchArticle(url)`
- Fetches page via CORS proxy (`https://corsproxy.io/?`)
- Parses HTML into DOM
- Extracts article using Readability
- Extracts published date
- Returns article object with title, date, and content

### `extractDate(doc, article)`
- Extracts published date from multiple sources (in order):
  1. `article.publishedTime` from Readability
  2. `<meta property="article:published_time">`
  3. `<meta name="date">`
  4. `<meta name="pubdate">`
  5. `<meta property="datePublished">`
  6. JSON-LD `datePublished` in `<script type="application/ld+json">`
  7. `<time datetime>` element
- Returns ISO date string or null

### `extractAuthor(doc, article)`
- Extracts author from multiple sources (in order):
  1. `article.byline` from Readability
  2. `<meta name="author">`
  3. `<meta property="article:author">`
  4. JSON-LD `author.name` in `<script type="application/ld+json">`
  5. `<a rel="author">` element
- Returns author string or null

### `slugify(text)`
- Converts text to URL-friendly slug
- Lowercase, spaces to hyphens, remove special characters
- Collapse multiple hyphens

### `buildMarkdown(article)`
- Builds markdown with key:value metadata at top
- Includes title, author, date, source (if available)
- Blank line separates metadata from content

### `buildFileCommand(area, title, markdown)`
- Generates bash/fish-compatible command
- Filename: `{area}.{slugified-title}.md`
- Uses heredoc with 'EOF' to handle special characters

Output format:
```
title: Article Title
author: John Doe
date: 2026-01-18
source: https://example.com/article

Article content...
```

### `convertToMarkdown(html)`
- Uses Turndown to convert HTML string to Markdown
- Preserves: bold, italic, headings (h1-h6), links, lists, blockquotes, code

### `selectAllContent(element)`
- Selects all text in the contenteditable div
- Enables immediate Ctrl+C by user

### `highlightMarkdown(code)`
- Applies Prism.js syntax highlighting for Markdown
- Returns highlighted HTML string

### Copy event handler
- Intercepts copy event on output element
- Ensures plain markdown text is copied (not highlighted HTML)

## Article Extraction Algorithm

Uses Mozilla's Readability.js (Firefox Reader View) which:

1. **Clones the DOM** - works on a copy to avoid mutations
2. **Removes unlikely candidates** - elements matching: `banner`, `sidebar`, `nav`, `footer`, `header`, `ad`, `comment`, `social`, `share`, `related`, `popup`, `modal`
3. **Scores candidates** - evaluates `<article>`, `<main>`, `<div>`, `<section>` by:
   - Paragraph count and text length
   - Text-to-link ratio (low link density = better)
   - Presence of semantic tags
4. **Selects top candidate** - highest scoring element
5. **Cleans result** - removes remaining noise

### False Positive Reduction

| Technique | What it catches |
|-----------|-----------------|
| Prefer `<article>` and `<main>` tags | Sites using semantic HTML |
| Penalize high link-density elements | Navigation menus, related links |
| Blacklist class/id patterns | Sidebars, ads, comments, social buttons |
| Minimum text length threshold | Prevents selecting tiny fragments |
| Ignore hidden elements | Modals, popups, lazy content |
| Text-to-markup ratio scoring | Favors content-heavy elements |

## User Workflows

### Rich Text Workflow
1. User copies text from a website (Ctrl+C)
2. User opens this tool
3. User pastes (Ctrl+V)
4. Markdown appears, fully selected
5. User copies (Ctrl+C)
6. Press Escape to reset

### URL Workflow
1. User copies article URL
2. User opens this tool
3. User pastes URL (Ctrl+V)
4. Loading indicator appears
5. Article is fetched and extracted
6. Markdown appears, fully selected
7. User copies (Ctrl+C)
8. Press Escape to reset
