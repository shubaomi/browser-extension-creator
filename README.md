# browser-extension-creator

A Claude Code skill for creating and maintaining cross-browser extensions (Chrome, Edge, Firefox, Safari) with a structured 5-step workflow.

## What is this?

This skill helps you build browser extensions from scratch. It provides:

- A complete workflow for creating browser extensions
- Templates and project structures for cross-browser compatibility
- Best practices for Manifest V3, service workers, content scripts
- Guidance on React/Vue popup UI integration
- Build and packaging instructions for distribution

## What can it create?

- Chrome extensions
- Edge extensions (Chromium-based, compatible with Chrome)
- Firefox WebExtensions
- Safari Web Extensions

Example extensions you can build:
- Bookmark organizers (auto-categorize by domain)
- GitHub helper tools (show repo stats, add features)
- Productivity tools (tab managers, note takers)
- Content scrapers and highlighters
- Any browser plugin you can imagine

## Installation

### Prerequisites

- [Claude Code](https://claude.ai/code) installed
- Node.js (for building with Vite)

### Install the skill

```bash
npx skills add https://github.com/shubaomi/browser-extension-creator.git
```

Or install from a local `.skill` file:

```bash
npx skills add /path/to/browser-extension-creator.skill
```

## Usage

After installation, invoke the skill by saying something like:

- "我想做一个浏览器插件"
- "帮我创建一个 Chrome 扩展"
- "I want to build a browser extension"
- "Create an extension that does X"

### Example prompts

**Simple:**
```
我想做一个浏览器插件，功能是自动把所有书签按照域名分类
```

**Detailed:**
```
I need a Chrome extension that:
- Shows GitHub repo stars and forks on repository pages
- Has a popup UI to toggle features
- Stores settings in browser storage
- Supports both Chrome and Firefox
```

## Workflow

The skill follows a 5-step workflow:

### Step 1: Discovery
Gather requirements — what the extension does, target browsers, permissions needed, UI components.

### Step 2: Architecture
Design the technical solution — manifest structure, project layout, cross-browser strategy.

### Step 3: Project Setup
Generate the project structure with templates for your chosen configuration.

### Step 4: Implementation
Write the extension code following best practices:
- Manifest V3 configuration
- Service worker / background scripts
- Content scripts
- Popup UI (React or Vue)
- Storage utilities

### Step 5: Build & Package
Build and package for distribution to Chrome Web Store, Firefox Add-ons, etc.

## Project Structure

For a cross-browser extension, the generated project looks like:

```
extension/
├── src/
│   ├── manifest/
│   │   ├── base.json      # Shared Manifest V3 config
│   │   ├── chrome.json     # Chrome-specific
│   │   └── firefox.json    # Firefox-specific
│   ├── background/
│   │   └── index.js        # Service worker
│   ├── content/
│   │   └── index.js        # Content script
│   ├── popup/
│   │   ├── popup.html
│   │   ├── popup.js
│   │   └── styles.css
│   └── shared/
│       ├── storage.ts      # Storage utilities
│       └── types.ts        # TypeScript types
├── icons/
├── build.js                # Build script
├── vite.config.ts          # Vite bundler config
└── package.json
```

## Key Features

### Cross-Browser Compatibility
- Uses `browser.*` namespace (Firefox/Safari standard)
- `webextension-polyfill` for Chrome/Edge compatibility
- Manifest layering strategy for browser-specific overrides

### Manifest V3
Always uses MV3 (Manifest Version 3), the current standard for all major browsers.

### Modern Popup UI
Supports React or Vue for popup interfaces, bundled with Vite.

### Best Practices Included
- Service worker lifecycle management
- Message passing between components
- Storage with proper error handling
- Security best practices (minimum permissions)

## Loading Your Extension

### Chrome / Edge
1. Go to `chrome://extensions/` or `edge://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select the `extension/` directory

### Firefox
1. Go to `about:debugging#/runtime/this-firefox`
2. Click "Load Temporary Add-on"
3. Select `manifest.json`

### Safari
1. Enable Developer menu in Safari → Preferences → Advanced
2. Click "Build Extension" in Developer menu

## Documentation

- [SKILL.md](SKILL.md) — Full skill documentation with all templates and patterns
- [evals/evals.json](evals/evals.json) — Test cases used to validate the skill

## Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## License

MIT
