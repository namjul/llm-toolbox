# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repo is a toolbox. Each tool is a self-contained single HTML file (SPA).

## Architecture

Each HTML file must be **directly runnable in a browser** - just open the file, no build step, no server, no extra setup required.

Constraints:

- **Tailwind CSS** via CDN (`@tailwindcss/browser@4`) for all styling
- **ES modules** via `importmap` for dependencies
- **esm.sh** for npm packages (e.g., `https://esm.sh/package@version`)
- **No TypeScript** - plain JavaScript only
- **Single file** - all HTML, CSS, and JS in one file
- **No script tags for JS libraries** - all JavaScript dependencies must be loaded via importmap, never via `<script src="...">` tags (CSS stylesheets via `<link>` are fine)

## Template Structure

```html
<!DOCTYPE html>
<html lang="en" class="h-full">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Tool Name</title>
  <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
</head>
<body class="p-10 bg-gray-50 space-y-3 h-full flex flex-col">
  <!-- UI here -->

  <script type="importmap">
  {
    "imports": {
      "library": "https://esm.sh/library@version"
    }
  }
  </script>

  <script type="module">
    // App logic here
  </script>
</body>
</html>
```

## Tool Index

`index.html` lists all tools in this repo. **When adding or removing a tool, always update index.html.**

Each tool entry includes:
- Link to the tool's HTML file
- Tool name
- Short description

## Tool Documentation (Bidirectional Mirror)

Each tool has a matching `.md` file that serves as the authoritative reference. For example, `paste-to-markdown.html` has `paste-to-markdown.md`.

**These files are bidirectional mirrors: a change in one implies a change in the other.**

The `.md` file contains:
- Purpose and description of the tool
- UI components and their behavior
- Dependencies (libraries used via importmap)
- Key functions and their responsibilities
- User interactions and workflows

**Keeping in Sync:**

Changes flow both directions:

| Change in `.html` | → Update `.md` |
|-------------------|----------------|
| Add UI element | Document in UI components section |
| Add dependency | Add entry with version and purpose |
| Add function | Document responsibility and behavior |
| Change behavior | Update workflow description |
| Remove functionality | Remove from documentation |

| Change in `.md` | → Update `.html` |
|-----------------|------------------|
| Document new UI element | Implement the element |
| Add dependency entry | Add to importmap and use it |
| Describe new function | Implement the function |
| Update workflow | Update code to match |
| Remove from docs | Remove from code |

## AI-Assisted Development

### Mandatory Planning Phase

For non-trivial tasks (new tools, refactors, significant changes), **respond with a plan BEFORE acting**:

1. **Understanding**: Restate the request in your own words
2. **TL;DR**: 2-5 bullet summary of approach and outcome
3. **Plan**: Step-by-step including affected files, assumptions, risks, alternatives

**No code changes until user confirms plan.**

Once confirmed, write/update the tool's `.md` file first, then implement the `.html` file.

**Scope**: Required for new tools and significant changes. Skip for trivial tasks (formatting, typos, scoped one-line changes).
