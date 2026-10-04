# Pi Configuration & Extensions

This directory contains pi configuration files and local extensions to sync across machines.

## Structure

```
pi/
├── settings.json              # Sample pi configuration
├── extensions/                # Local extensions for ~/.pi/agent/extensions
│   ├── ask-user-question.ts  # Enhanced ask-user-question tool with TUI editor
│   ├── custom-header.ts      # Custom startup header
│   ├── browser/              # Headless Playwright browser tools
│   ├── web-fetch/            # URL/PDF to markdown converter
│   └── prompt-snippets/      # Toggle prompt rules with alt+s
└── README.md                 # This file
```

## Installation

### 1. Install pi packages

```bash
# Already covered in Phase 5.7 of Linux.md/Mac.md
pi install npm:pi-9router-ext
pi install npm:pi-subagents
pi install npm:pi-typesafe
```

### 2. Configure pi settings

Copy or merge `settings.json` into `~/.pi/agent/settings.json`:

```bash
# Backup existing settings
cp ~/.pi/agent/settings.json ~/.pi/agent/settings.json.backup

# Option 1: Copy sample (overwrites)
cp pi/settings.json ~/.pi/agent/settings.json

# Option 2: Merge manually
# Edit ~/.pi/agent/settings.json and add the keys from pi/settings.json
```

**Important settings:**
- `defaultProvider`: Set to `"9router"` if using 9router
- `defaultModel`: Your preferred model (e.g., `"gap/claude-opus-5-5"`)
- `defaultThinkingLevel`: `"high"`, `"medium"`, or `"low"`
- `hideThinkingBlock`: `true` to hide thinking blocks in output

### 3. Install local extensions

```bash
# Copy all extensions
cp -r pi/extensions/* ~/.pi/agent/extensions/

# Or use symlinks for easier updates
ln -sf "$(pwd)/pi/extensions/ask-user-question.ts" ~/.pi/agent/extensions/
ln -sf "$(pwd)/pi/extensions/custom-header.ts" ~/.pi/agent/extensions/
ln -sf "$(pwd)/pi/extensions/browser" ~/.pi/agent/extensions/
ln -sf "$(pwd)/pi/extensions/web-fetch" ~/.pi/agent/extensions/
ln -sf "$(pwd)/pi/extensions/prompt-snippets" ~/.pi/agent/extensions/
```

### 4. Install extension dependencies

**browser extension** (requires Playwright Chromium):
```bash
cd ~/.pi/agent/extensions/browser
npm install
npx playwright install chromium
```

**web-fetch extension**:
```bash
cd ~/.pi/agent/extensions/web-fetch
npm install
```

### 5. Enable TypeSafe (optional)

If you installed `pi-typesafe`:

```bash
# In pi, run:
/typesafe enable
```

## Extensions Overview

| Extension | Purpose | Dependencies |
|-----------|---------|--------------|
| **ask-user-question.ts** | Ask-the-user tool with TUI editor for multi-line input | None (built-in) |
| **custom-header.ts** | Customize the pi startup header | None (built-in) |
| **browser/** | Headless browser automation (goto, eval, fill, click, screenshot) | playwright-core |
| **web-fetch/** | Fetch URLs and convert HTML/PDF to markdown | @mozilla/readability, turndown, unpdf |
| **prompt-snippets/** | Toggle prompt rules with alt+s (6 snippets: ask-questions, delegate-exploration, diagnose-report, orchestrator-mode, session-kickoff, verify-not-assume) | None (built-in) |

## Updating

To sync changes from this repo to your local pi installation:

```bash
# Pull latest changes
git pull

# Re-copy or update symlinks
cp -r pi/extensions/* ~/.pi/agent/extensions/
# or if using symlinks, they auto-update

# Update extension dependencies if needed
cd ~/.pi/agent/extensions/browser && npm install
cd ~/.pi/agent/extensions/web-fetch && npm install
```

## Notes

- **web-fetch vs ninerouter_web_fetch**: Both fetch URLs. `web-fetch` is local (no external service), while `ninerouter_web_fetch` uses 9router. Choose one based on your setup.
- **prompt-snippets**: Use alt+s in pi to toggle snippets on/off. Active snippets are prepended/appended to your messages.
- **custom-header.ts**: Edit the `buildHeader()` function to customize your startup message.
- **ask-user-question.ts**: Enhances the built-in ask_user_question tool with a full-screen TUI editor.

## Troubleshooting

**Extensions not loading:**
```bash
# Check extension directory
ls -la ~/.pi/agent/extensions/

# Check pi logs
pi # then check startup messages
```

**Browser extension fails:**
```bash
# Reinstall Chromium
cd ~/.pi/agent/extensions/browser
npx playwright install chromium
```

**Dependency errors:**
```bash
# Reinstall deps
cd ~/.pi/agent/extensions/browser  # or web-fetch
rm -rf node_modules package-lock.json
npm install
```
