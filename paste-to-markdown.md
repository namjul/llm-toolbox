# Paste to Markdown

A tool to convert rich text or web articles into Markdown format.

## Purpose

Two input modes:
1. **Rich text**: Copy formatted text from a website, paste here, get Markdown
2. **URL**: Paste an article URL, tool fetches and extracts the main content as Markdown

Supported URL sources:
- **Twitter/X.com**: Tweets via oEmbed API
- **Substack**: Articles via preloaded JSON data
- **General websites**: Articles via Mozilla Readability

## Output Modes

### Copy Mode (default)
- Output: Markdown text, auto-selected for copying

### Command Mode
- Output: bash/fish command to paste into CLI that creates a file
- Filename auto-generated from title as `{slugified-title}.md`, or current date if no title
- User can edit the filename directly in the input field
- Optional prefix with variable support (persisted in localStorage)

#### Prefix Feature
- Checkbox to enable/disable prefix (persisted, default: disabled)
- Prefix template input (persisted, defaults to `{author}` if empty)
- Output format: `{resolved-prefix}.{filename}.md` or `{filename}.md` if disabled

#### Supported Variables
| Variable | Description |
|----------|-------------|
| `{author}` | Slugified author name |
| `{title}` | Slugified article title |
| `{date}` | Article date as YYYY.MM.DD |
| `{year}` | Article year (4 digits) |
| `{month}` | Article month (2 digits) |
| `{day}` | Article day (2 digits) |
| `{domain}` | Slugified source domain (without www) |

Example output (with prefix `{author}`):
```bash
printf '%s' 'title: How to Build a Web App
author: John Doe
date: 2026-01-18
source: https://example.com

Content here...' > "john-doe.how-to-build-a-web-app.md"
```

Example output (without prefix):
```bash
printf '%s' 'title: How to Build a Web App
author: John Doe
date: 2026-01-18
source: https://example.com

Content here...' > "how-to-build-a-web-app.md"
```

## UI Components

### Header Bar
- Mode toggle: Copy / Command
- Prefix checkbox (visible in command mode): enables/disables prefix, persisted in localStorage
- Prefix input (visible when checkbox enabled): template with variable support, persisted in localStorage
- Filename input (visible in command mode): shows auto-generated base filename (placeholder shows default), editable by user, `.md` extension displayed as static suffix
- On blur of prefix/filename inputs, re-selects output for copying

### State 1: Empty (Initial)
- Full-page paste target
- Instructions text: "Paste content or URL (Ctrl+V)"

### State 2: Loading (URL only)
- Loading indicator while fetching and parsing URL

### State 3: Result
- Contenteditable div displaying the output with syntax highlighting
- All text automatically selected for immediate copy (Ctrl+C)
- Auto-selection triggers on: new paste, mode change, or input blur (prefix/filename)
- Inputs can be edited without losing focus to the output
- Press Escape to reset to State 1

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| spellcaster | 6.0.0 | Reactive state management and hyperscript |
| turndown | 7.1.2 | Convert HTML to Markdown |
| @mozilla/readability | 0.5.0 | Extract article content from web pages |
| prismjs | 1.29.0 | Markdown/bash syntax highlighting |

## Key Functions

### `handlePaste(event)`
- Intercepts paste event
- Detects if content is a URL or rich text
- Routes to appropriate handler

### `isUrl(text)`
- Returns true if text matches URL pattern (http/https)

### `fetchArticle(url)`
- Tries platform-specific extraction in order:
  1. Twitter/X.com via oEmbed API (no CORS proxy needed)
  2. Substack via `window._preloads` JSON
  3. Generic extraction via Readability
- Fetches page via CORS proxy (`https://corsproxy.io/?`) for generic sites
- Extracts published date and author
- Returns article object with title, date, author, and content

### `isTwitterUrl(url)`
- Returns true if URL is from x.com or twitter.com

### `fetchTwitterContent(url)`
- Fetches tweet via Twitter's oEmbed API (`publish.twitter.com/oembed`)
- Extracts tweet text from the returned HTML blockquote
- Returns object with content, author, title ("Tweet by {author}"), and source

### `extractSubstackContent(html)`
- Extracts article from Substack's `window._preloads` JSON
- Returns title, date, author, and body_html
- Returns null for non-Substack pages

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

### `buildFileCommand(prefix, filename, markdown)`
- Generates bash/fish-compatible command
- Combines prefix and filename: `{prefix}.{filename}.md` or `{filename}.md` if no prefix
- Uses printf with escaped single quotes for shell compatibility

### `generateFilename(title)`
- Generates default base filename from title (without extension)
- Returns `{slugified-title}` if title exists
- Falls back to current date `{YYYY.MM.DD}` if no title

### `resolvePrefix(template, article)`
- Replaces variables in prefix template with article metadata
- Supported variables: `{author}`, `{title}`, `{date}`, `{year}`, `{month}`, `{day}`, `{domain}`
- Returns empty string if template is empty
- Unrecognized variables are left unchanged

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
