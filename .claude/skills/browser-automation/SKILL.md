---
name: Browser Automation
description: Automate web browser interactions using natural language via CLI commands. Use when the user asks to browse websites, navigate web pages, extract data from websites, take screenshots, fill forms, click buttons, or interact with web applications. Triggers include "browse", "navigate to", "go to website", "extract data from webpage", "screenshot", "web scraping", "fill out form", "click on", "search for on the web". When taking actions be as specific as possible.
allowed-tools: Bash
---

# Browser Automation

Automate browser interactions using Stagehand CLI with Claude. This skill provides natural language control over a Chrome browser through command-line tools for navigation, interaction, data extraction, and screenshots.

## Overview

This skill uses a CLI-based approach where Claude Code calls browser automation commands via bash. The browser stays open between commands for faster sequential operations and preserves browser state (cookies, sessions, etc.).

## Setup Verification

**IMPORTANT: Before using any browser commands, you MUST check setup.json in this directory.**

### First-Time Setup Check

1. **Read `setup.json`** (located in `.claude/skills/browser-automation/setup.json`)
   - If the file doesn't exist, copy from template: `cp setup.example.json setup.json`
2. **Check `setupComplete` field**:
   - If `true`: All prerequisites are met, proceed with browser commands
   - If `false`: Setup required - follow the steps below

### If Setup is Required (`setupComplete: false`)

Run these commands in the plugin directory:

```bash
# 1. Install dependencies and build (REQUIRED)
# This automatically builds TypeScript
npm install
# or: pnpm install
# or: bun install

# 2. Link the browser command globally (REQUIRED)
npm link

# 3. Configure API key (REQUIRED)
# Option 1 (RECOMMENDED): Export in your terminal
export ANTHROPIC_API_KEY="your-api-key-here"

# Option 2: Or use .env file
cp .env.example .env
# Then edit .env and add: ANTHROPIC_API_KEY="your-api-key-here"

# 4. Verify Chrome is installed
# Chrome should be at standard location for your OS

# 5. Test the installation
browser navigate https://example.com

# 6. If test succeeds, update setup.json
# Set all "installed"/"configured" fields to true
# Set "setupComplete" to true
```

### Prerequisites Summary

- ✅ Google Chrome installed on your system
- ✅ Node.js dependencies installed and TypeScript built (`npm install` runs build automatically)
- ✅ Browser command globally available (`npm link` creates the global symlink)
- ✅ Anthropic API key configured (exported as `ANTHROPIC_API_KEY` environment variable or in `.env` file)

**DO NOT attempt to use browser commands if `setupComplete: false` in setup.json. Guide the user through setup first.**

## Available Commands

### Navigate to URLs
```bash
browser navigate <url>
```

**When to use**: Opening any website, loading a specific URL, going to a web page.

**Example usage**:
- `browser navigate https://example.com`
- `browser navigate https://news.ycombinator.com`

**Output**: JSON with success status, message, and screenshot path

### Interact with Pages
```bash
browser act "<action>"
```

**When to use**: Clicking buttons, filling forms, scrolling, selecting options, typing text.

**Example usage**:
- `browser act "click the Sign In button"`
- `browser act "fill in the email field with test@example.com"`
- `browser act "scroll down to the footer"`
- `browser act "type 'laptop' in the search box and press enter"`

**Important**: Be as specific as possible - details make a world of difference. When filling fields, you don't need to combine 'click and type'; the tool will perform a fill similar to Playwright's fill function.

**Output**: JSON with success status, message, and screenshot path

### Extract Data
```bash
browser extract "<instruction>" ['{"field": "type"}']
```

**When to use**: Scraping data, getting specific information, collecting structured content.

**Schema format** (optional): JSON object where keys are field names and values are types:
- `"string"` for text
- `"number"` for numeric values
- `"boolean"` for true/false values

**Note**: The schema parameter is optional. If omitted or if schema validation fails, extraction will proceed without type validation.

**Example usage**:
- `browser extract "get the product title and price" '{"title": "string", "price": "number"}'`
- `browser extract "get all article headlines" '{"headlines": "string"}'`
- `browser extract "get the page title"` (no schema)

**Output**: JSON with success status, extracted data, and screenshot path

### Discover Elements
```bash
browser observe "<query>"
```

**When to use**: Understanding page structure, finding what's clickable, discovering form fields.

**Example usage**:
- `browser observe "find all clickable buttons"`
- `browser observe "find all form fields"`
- `browser observe "find all navigation links"`

**Output**: JSON with success status, discovered elements, and screenshot path

### Take Screenshots
```bash
browser screenshot
```

**When to use**: Visual verification, documenting page state, debugging, creating records.

**Notes**:
- Screenshots are saved to the plugin directory's `agent/browser_screenshots/` folder
- Images larger than 2000x2000 pixels are automatically resized
- Filename includes timestamp for uniqueness

**Output**: JSON with success status and screenshot path

### Clean Up
```bash
browser close
```

**When to use**: After completing all browser interactions, to free up resources.

**Output**: JSON with success status and message

## Browser Behavior

**Persistent Browser**: The browser stays open between commands for faster sequential operations and to preserve browser state (cookies, sessions, etc.).

**Reuse Existing**: If Chrome is already running on the profile's CDP port, it will reuse that instance.

**Minimized Launch**: Chrome opens off-screen (position -9999,-9999) to avoid disrupting workflow.

**Safe Cleanup**: The browser only closes when you explicitly call the `close` command.

## Chrome Profile Support

This fork supports multiple Chrome profiles via the `BROWSER_PROFILE` environment variable. Each profile uses a separate CDP port to allow simultaneous operation.

### Environment Variable

Set `BROWSER_PROFILE` to specify which Chrome profile to use:

```bash
# Use default profile
browser navigate https://example.com

# Use a specific profile
BROWSER_PROFILE="Profile 2" browser navigate https://example.com
```

### Profile Directories

Chrome stores profiles in directories like:
- `Default` - The default profile
- `Profile 1`, `Profile 2`, etc. - Additional profiles

To find your profiles and their associated accounts:
```bash
# macOS
for p in Default "Profile 1" "Profile 2"; do
  echo "=== $p ==="
  jq -r '.account_info[0].email // "(not signed in)"' \
    "$HOME/Library/Application Support/Google/Chrome/$p/Preferences" 2>/dev/null
done
```

### Creating Wrapper Scripts

For convenience, create wrapper scripts in `/usr/local/bin/` for each profile.
Example wrapper scripts are provided in the `wrappers/` directory within this skill.

To install them:

```bash
# Copy wrappers to /usr/local/bin
sudo cp wrappers/browser wrappers/browser-work wrappers/browser-home /usr/local/bin/
sudo chmod +x /usr/local/bin/browser /usr/local/bin/browser-work /usr/local/bin/browser-home

# Edit each to set your ANTHROPIC_API_KEY and BROWSER_PROFILE
```

See `wrappers/README.md` for detailed instructions on finding your Chrome profiles.

### Cache Directories

Profiles are copied on first use to avoid modifying your real Chrome data:
- Default profile: `.chrome-profile/`
- Other profiles: `.chrome-profile-{profile-name}/` (e.g., `.chrome-profile-profile-2/`)

To force a fresh profile copy, delete the relevant `.chrome-profile*` directory.

### CDP Ports

Each profile uses a different Chrome DevTools Protocol port:
- Default: 9222
- Other profiles: 9223-9322 (based on profile name hash)

This allows running multiple profiles simultaneously without conflicts.

## Best Practices

1. **Always navigate first**: Before interacting with a page, navigate to the URL
2. **📸 Always view screenshots**: After each command (navigate, act, extract, observe), use the Read tool to view the screenshot and verify the command worked correctly
3. **Use natural language**: Describe actions as you would instruct a human
4. **Extract with clear schemas**: Define field names and types explicitly in JSON
5. **Handle errors gracefully**: Check the `success` field in JSON output; if an action fails, view the screenshot and try using `observe` to understand the page better
6. **Close when done**: Always clean up browser resources after completing tasks
7. **Be specific**: Use precise selectors in natural language ("the blue Submit button" vs "the button")
8. **Chain commands**: Run multiple commands sequentially without reopening the browser

## Common Patterns

### Simple browsing task
```bash
browser navigate https://example.com
browser act "click the login button"
browser screenshot
browser close
```

### Data extraction task
```bash
browser navigate https://example.com/products
browser act "wait for page to load"
browser extract "get all products" '{"name": "string", "price": "number"}'
# Or without schema:
# browser extract "get the page content"
browser close
```

### Multi-step interaction
```bash
browser navigate https://example.com/login
browser act "fill in email with user@example.com"
browser act "fill in password with mypassword"
browser act "click the submit button"
browser screenshot
browser close
```

### Debugging workflow
```bash
browser navigate https://example.com
browser screenshot
browser observe "find all buttons"
browser act "click the specific button"
browser screenshot
browser close
```

## Troubleshooting

**Page not loading**: Wait a few seconds after navigation before acting. You can explicitly: `browser act "wait for the page to fully load"`

**Element not found**: Use `observe` to discover what elements are actually available on the page

**Action fails**: Be more specific in natural language description. Instead of "click the button", try "click the blue Submit button in the form"

**Screenshots missing**: Check the plugin directory's `agent/browser_screenshots/` folder for saved files

**Chrome not found**: Install Google Chrome or the CLI will show an error with installation instructions

**Port 9222 in use**: Another Chrome debugging session is running. Close it or wait for timeout

**First-run profile copy race condition**: When a profile is copied for the first time, you may see `Cannot read properties of undefined (reading 'page')`. This is a known race condition where the profile copy completes but Stagehand's page initialization races. Simply retry the command - subsequent attempts will succeed since the profile is already copied.

**Stale Chrome instance**: If commands fail with the "page undefined" error even after retry, a Chrome process may be holding the CDP port but disconnected from Stagehand. Kill it and retry:
```bash
# For default profile (port 9222)
lsof -ti :9222 | xargs kill

# For other profiles, check which port (e.g., 9226 for Profile 2)
lsof -ti :9226 | xargs kill
```

For detailed examples, see [EXAMPLES.md](EXAMPLES.md).
For API reference and technical details, see [REFERENCE.md](REFERENCE.md).

## Dependencies

To use this skill, install these dependencies only if they aren't already present:

```bash
npm install
# or
pnpm install
# or
bun install
```
