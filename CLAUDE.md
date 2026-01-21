# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repo is a toolbox. Each tool is a self-contained single HTML file (SPA) built mostly with the help of LLMs.

## Architecture

Each HTML file must be **directly runnable in a browser** - just open the file, no build step, no server, no extra setup required.

Constraints:

- ⛔ **HTML/MD sync is mandatory** - NEVER edit a `.html` file without updating its `.md` file in the SAME response, and vice versa. No exceptions.
- **Tailwind CSS** via CDN (`@tailwindcss/browser@4`) for all styling
- **ES modules** via `importmap` for dependencies
- **esm.sh** for npm packages (e.g., `https://esm.sh/package@version`)
- **No TypeScript** - plain JavaScript only
- **Single file** - all HTML, CSS, and JS in one file
- **No script tags for JS libraries** - all JavaScript dependencies must be loaded via importmap, never via `<script src="...">` tags (CSS stylesheets via `<link>` are fine)
- **Zag.js for complex UI components** - prefer [Zag.js](https://zagjs.com/) for complex accessible components (dialogs, menus, comboboxes) when the accessibility requirements justify the setup. For simple controls like toggles or button groups, Spellcaster signals with native elements are sufficient and simpler. Note: Zag.js vanilla JS usage requires custom infrastructure not exported from the package.

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

### ⛔ HARD RULE: Mandatory Sync

**NEVER edit a `.html` file without updating its `.md` file in the SAME response.**
**NEVER edit a `.md` file without updating its `.html` file in the SAME response.**

This is NON-NEGOTIABLE. No exceptions. No "I'll do it later." No waiting to be asked.

- Change `.html` → update `.md` to document the new behavior
- Change `.md` → update `.html` to implement the documented behavior

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

### Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/) with the tool name as scope:

```
type(tool-name): description
```

| Type | Usage |
|------|-------|
| `feat` | New feature or functionality |
| `fix` | Bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `docs` | Documentation only (CLAUDE.md, README) |
| `chore` | Maintenance (dependencies, config) |

Examples:
- `feat(paste-to-markdown): add Twitter/X.com support`
- `fix(paste-to-markdown): handle empty URL edge case`
- `refactor(paste-to-markdown): use Spellcaster v6 idioms`
- `docs: strengthen HTML/MD sync rule` (no scope for project-wide docs)
