# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development

```bash
# Build TypeScript (required after code changes)
npm run build

# Run interactive agent directly (development mode, no build needed)
npm run claude

# The browser CLI is globally linked - after building, commands work immediately:
browser navigate https://example.com
browser act "click the button"
browser close
```

**Key point**: The global `browser` command symlinks to `dist/src/cli.js`. After editing TypeScript in `src/`, run `npm run build` to see changes. No re-linking needed.

## Architecture

This is a Claude Code plugin that provides browser automation via Stagehand (AI browser automation built on Playwright).

### Two Entry Points

1. **CLI Tool** (`src/cli.ts` → `dist/src/cli.js`)
   - Compiled to global `browser` command via npm link
   - Used by Claude Code skill to execute browser actions
   - Spawns Chrome with CDP, connects via Stagehand, executes single command, exits
   - Chrome persists between CLI invocations (PID tracked in `.chrome-pid`)

2. **Interactive Agent** (`agent-browse.ts`)
   - Development/testing tool using Claude Agent SDK
   - Multi-turn conversation with streaming
   - Calls the CLI tool via bash for browser actions

### Core Modules

- `src/browser-utils.ts` - Chrome path detection, profile management, screenshot capture
- `src/cli.ts` - Main CLI: browser init, commands (navigate/act/extract/observe/screenshot/close), cleanup
- `src/network-monitor*.ts` - Network traffic monitoring utilities (development tools)

### Chrome Profile Handling

- Copies user's Chrome profile to `.chrome-profile/` (or `.chrome-profile-{profile-name}/`) on first run
- `BROWSER_PROFILE` env var selects which Chrome profile to use
- Each profile gets a unique CDP port (9222 for Default, 9223-9322 for others)

### Plugin Structure

```
.claude/skills/browser-automation/SKILL.md  # Skill definition for Claude Code
agent/browser_screenshots/                   # Screenshot output directory
agent/downloads/                             # Downloaded files directory
```
