# Auto-Achievo

CLI tool to sync timewarrior entries to the Achievo time tracking system.

**Repository:** [github.com/namjul/auto-achievo](https://github.com/namjul/auto-achievo)
**Type:** Node.js CLI (TypeScript)

## Purpose

Eliminates manual time entry by reading timewarrior output, mapping tags to Achievo fields via configuration, and automating the browser to fill in forms.

## Features

### Timewarrior Parsing

- Parses `timew summary` output format with date, ID, tags, annotation, start/end times
- Handles continuation lines (entries without date inherit from previous)
- Detects and rejects active (still running) entries with clear error message
- Reads from stdin (piped) or file via `--file` flag

### Tag-Based Mapping

- YAML configuration maps timewarrior tags to Achievo projects, phases, and activities
- **Project tags are required** - each entry must have exactly one project tag (e.g., `+emt`)
- Error if entry has no project tag or multiple conflicting project tags
- Phase and activity fall back to `default` if no specific tag matches
- `@`-prefixed tags are organizational (filtered out, not used for mapping or comments)
- Unused tags (not used for mapping) are included in the Achievo remark field

### Smart Rounding

- Rounds durations to configurable interval (default 15 minutes)
- **Per-project totals are preserved** using the largest remainder method
- Entries under 5 minutes are dropped, but their time is redistributed to other entries in the same project
- Prevents systematic time loss/gain from naive rounding

### Summary Display

- Shows all entries grouped by date before submission
- Displays which tag matched each field in parentheses (e.g., `EMT Helpdesk (+phase-help)`)
- Shows daily totals and grand total
- Dry-run mode (`--dry-run`) shows summary without submitting

### Browser Automation

- Uses Playwright with Chromium
- **Browser visible by default** so user can log in manually
- Waits up to 5 minutes for user to complete login
- Handles Achievo's iframe-based UI (version 1.4.6)
- **Cascading dropdown handling** - project/phase selections trigger page reloads; tool waits for reload and re-sets values that get reset by server
- **Fuzzy option matching** - tries exact match first, falls back to substring match, handles `&nbsp;` in option labels
- 1 second delay between entries to avoid overwhelming the server

### CLI Options

| Flag | Description |
|------|-------------|
| `--dry-run`, `-d` | Preview without submitting |
| `--headless`, `-H` | Hide browser window |
| `--file <path>`, `-f` | Read from file instead of stdin |
| `--config <path>`, `-c` | Config file (default: `config.yaml`) |

## Configuration

YAML file with:

- `url` - Achievo instance URL
- `roundingInterval` - Minutes (default 15)
- `project` - Map of project tags to project config
  - Each project has `name`, `phase` mappings, and `activity` mappings
  - Use `default` key for fallback values within phase/activity

## Module Responsibilities

| Module | Purpose |
|--------|---------|
| `parser.ts` | Parse timewarrior text format, detect active entries |
| `mapper.ts` | Load YAML config, map tags to Achievo fields, track which tags were used |
| `aggregator.ts` | Smart rounding per project, build comments from unused tags, format summary |
| `automation/login.ts` | Launch browser, detect login state, wait for manual authentication |
| `automation/timeEntry.ts` | Navigate to hours page, fill form fields, handle cascading dropdowns |

## Dependencies

- **commander** - CLI argument parsing
- **playwright** - Browser automation
- **yaml** - Configuration parsing
- **typescript** - Development
