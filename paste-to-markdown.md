# Paste to Markdown

A tool to convert rich text (copied from websites) into Markdown format.

## Purpose

When you copy text from a website article, it includes HTML formatting (bold, italic, headings, links, lists). This tool converts that rich text into clean Markdown that can be saved to a `.md` file.

## UI Components

### State 1: Empty (Initial)
- Full-page paste target
- Instructions text: "Paste your content (Ctrl+V)"

### State 2: Pasted (Result)
- Contenteditable div displaying the converted Markdown with syntax highlighting
- All text automatically selected for immediate copy (Ctrl+C)
- Press Escape to reset to State 1

## Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| turndown | latest | Convert HTML to Markdown |
| spellcaster | latest | Reactive state management |
| prism.js | latest | Markdown syntax highlighting |

## Key Functions

### `handlePaste(event)`
- Intercepts paste event
- Extracts HTML from clipboard via `event.clipboardData.getData('text/html')`
- Falls back to plain text if no HTML available

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

## User Workflow

1. User copies text from a website (Ctrl+C)
2. User opens this tool
3. User pastes (Ctrl+V) - paste works anywhere on page
4. Markdown appears, fully selected
5. User copies (Ctrl+C)
6. User pastes into their markdown file
7. Click or keypress to reset and paste again
