# Browser Wrapper Scripts

Example wrapper scripts for using different Chrome profiles with the browser CLI.

## Prerequisites

1. Install agent-browse dependencies: `npm install`
2. Link the browser command globally: `npm link`

## Installation

1. Copy the wrapper scripts to `/usr/local/bin/`:

```bash
sudo cp browser browser-work browser-home /usr/local/bin/
sudo chmod +x /usr/local/bin/browser /usr/local/bin/browser-work /usr/local/bin/browser-home
```

2. Edit each script to set:
   - `ANTHROPIC_API_KEY` - Your Anthropic API key
   - `BROWSER_PROFILE` - The Chrome profile directory to use

## Finding Your Chrome Profiles

Chrome stores profiles in directories like `Default`, `Profile 1`, `Profile 2`, etc.

To see which email is associated with each profile:

```bash
# macOS
for p in Default "Profile 1" "Profile 2" "Profile 3"; do
  email=$(jq -r '.account_info[0].email // "(not signed in)"' \
    "$HOME/Library/Application Support/Google/Chrome/$p/Preferences" 2>/dev/null)
  [ -n "$email" ] && echo "$p: $email"
done
```

## Usage

```bash
# Default profile
browser navigate https://example.com

# Work profile
browser-work navigate https://github.com

# Home profile
browser-home navigate https://gmail.com
```

## API Key

You can set the API key in three ways (in order of precedence):

1. Edit it directly in the wrapper script
2. Export it in your shell: `export ANTHROPIC_API_KEY="sk-ant-..."`
3. Create a `.env` file in the agent-browse directory
