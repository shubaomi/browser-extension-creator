---
name: browser-extension-creator
description: |
  Browser extension creation and maintenance skill. Use whenever the user wants to create a new browser extension/plugin, modify an existing extension, or maintain cross-browser extension projects.
  
  Triggers include:
  - "我想做一个浏览器插件"
  - "帮我创建一个Chrome扩展"
  - "browser extension", "浏览器插件", "浏览器扩展"
  - "我想做个插件功能是..."
  - "帮我做一个XX插件"
  - Questions about extension manifest.json, background scripts, content scripts, popup UI
  - Cross-browser extension development (Chrome, Edge, Firefox, Safari)
  - Manifest V2 to V3 migration
  - Extension build system setup

  This skill provides templates, workflows, and code generation for building browser extensions with React/Vue popup UI support.
---

# Browser Extension Creator

Create and maintain cross-browser extensions with a structured workflow. Supports Chrome, Edge, Firefox, and Safari.

## Core Principle

**Start with understanding, not code.** A well-designed extension starts with clear requirements. This skill guides you through a discovery process before generating any code.

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Discovery          Understand the extension       │
│  Step 2: Architecture       Design the technical solution  │
│  Step 3: Project Setup      Generate project structure      │
│  Step 4: Implementation    Write the extension code        │
│  Step 5: Build & Package    Build for distribution          │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 1: Discovery

Before writing any code, gather requirements. Ask the user:

### 1.1 Extension Purpose
- **What does the extension do?** (1-2 sentence summary)
- **Who is the target user?**
- **What problem does it solve?**

### 1.2 Browser Support
Which browsers need to be supported?
- Chrome ✅ (primary target, most common)
- Edge ✅ (Chromium-based, highly compatible with Chrome)
- Firefox ✅ (good WebExtension support)
- Safari ✅ (Safari Web Extension, macOS only)

**Recommendation:** Default to Chrome + Edge as primary targets (Chromium-based, ~85% market share), Firefox as secondary.

### 1.3 Technical Requirements

#### Permissions Needed
What browser APIs does the extension need?
- `tabs` — interact with browser tabs
- `storage` — store settings/data
- `bookmarks` — read/write bookmarks (organize-bookmarks use case)
- `activeTab` — access current tab content
- `downloads` — manage downloads
- `notifications` — system notifications
- `contextMenus` — right-click menu items
- `webNavigation` — track page navigation
- Host permissions — specific sites or `<all_urls>`

#### Popup UI
- **Does the extension need a popup?** (user clicks extension icon)
  - If yes: React or Vue?
  - If no: consider toolbar action or options page

#### Background Behavior
- **Does it need background scripts/service workers?**
  - Periodic tasks (alarms, interval checks)
  - Cross-tab communication
  - Long-running operations

#### Content Scripts
- **Does it need to run on web pages?**
  - Inject into specific sites or all sites?
  - Need to modify page DOM or read page content?

### 1.4 Data Storage
- `browser.storage.local` — device-local storage (~10MB)
- `browser.storage.sync` — cross-device sync (~100KB, Google account needed)
- `browser.storage.session` — session-only, cleared on browser close

### 1.5 Discovery Template

Use this template to capture requirements:

```markdown
## Extension Specification

**Name:** [Extension name]
**Summary:** [1-2 sentences describing what it does]
**Target Users:** [Who will use this]
**Problem Solved:** [The pain point it addresses]

### Browser Support
- [ ] Chrome
- [ ] Edge
- [ ] Firefox
- [ ] Safari

### Core Features (Priority Order)
1. [Most important feature]
2. [Second most important]
3. [Nice to have]

### Permissions Required
- [ ] storage
- [ ] tabs
- [ ] bookmarks
- [ ] activeTab
- [ ] <list others>

### UI Components
- [ ] Popup (click extension icon)
- [ ] Options Page (right-click → Options)
- [ ] Context Menu
- [ ] Badge/Overlay on page

### Data Storage
- [ ] browser.storage.local
- [ ] browser.storage.sync
- [ ] browser.storage.session

### Background Processing
- [ ] Service Worker / Background Script
- [ ] Periodic tasks (alarms API)
- [ ] Native messaging

### Content Scripts
- [ ] Inject into all pages
- [ ] Inject into specific sites: [list sites]
```

---

## Step 2: Architecture

Based on requirements, design the technical architecture.

### 2.1 Manifest Strategy

**Manifest V3 (MV3)** is the current standard for all browsers. Always use MV3 unless there's a specific reason not to.

```
manifest/
├── manifest.base.json        # Shared config
├── manifest.chrome.json     # Chrome-specific overrides
├── manifest.firefox.json     # Firefox-specific overrides
├── manifest.edge.json        # Edge-specific overrides
├── manifest.safari.json       # Safari-specific overrides
```

**Base manifest structure:**
```json
{
  "manifest_version": 3,
  "name": "__MSG_extName__",
  "version": "1.0",
  "description": "__MSG_extDescription__",
  "default_locale": "en",
  "icons": {
    "48": "icons/icon-48.png",
    "96": "icons/icon-96.png",
    "128": "icons/icon-128.png"
  },
  "permissions": [],
  "host_permissions": [],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup/popup.html",
    "default_icon": {
      "48": "icons/icon-48.png",
      "96": "icons/icon-96.png"
    }
  }
}
```

### 2.2 Project Structure

Choose one of two structures based on complexity:

#### Simple Structure (single browser or Chrome-only)
```
extension/
├── manifest.json
├── background.js
├── content/
│   └── script.js
├── popup/
│   ├── popup.html
│   ├── popup.js
│   └── styles.css
├── icons/
│   ├── icon-48.png
│   ├── icon-96.png
│   └── icon-128.png
└── options/
    └── options.html
```

#### Monorepo Structure (cross-browser)
```
extension-project/
├── packages/
│   ├── common/              # Shared code
│   │   ├── src/
│   │   │   ├── content/
│   │   │   ├── background/
│   │   │   ├── popup/
│   │   │   └── shared/
│   │   └── package.json
│   │
│   ├── chrome/              # Chrome build output
│   ├── firefox/             # Firefox build output
│   ├── edge/                # Edge build output
│   └── safari/             # Safari build output
│
├── src/                     # Source files
│   ├── background/
│   │   └── index.ts
│   ├── content/
│   │   └── index.ts
│   ├── popup/
│   │   ├── index.html
│   │   ├── App.tsx          # React component
│   │   └── main.tsx
│   ├── shared/
│   │   ├── storage.ts
│   │   └── messages.ts
│   └── manifest/
│       ├── base.json
│       ├── chrome.json
│       ├── firefox.json
│       └── edge.json
│
├── icons/                   # Source icons (SVG preferred)
├── build.js                 # Build script
├── vite.config.ts           # Bundler config
└── package.json
```

### 2.3 Cross-Browser Compatibility

**Key strategy: Use `browser.*` namespace (Firefox/Safari standard) + webextension-polyfill for Chrome/Edge.**

```javascript
// In background.js or content script
// Import polyfill at the top of every entry point
import browser from 'webextension-polyfill';

browser.runtime.sendMessage({ action: "hello" });
browser.storage.local.get("key");
```

**In manifest.json background scripts:**
```json
"background": {
  "scripts": ["browser-polyfill.js", "background.js"],
  "type": "module"
}
```

### 2.4 Technology Choices

#### Popup UI: React vs Vue

| | React | Vue |
|---|---|---|
| Bundle size | ~40KB | ~33KB |
| Learning curve | Steeper | Gentler |
| Ecosystem | Larger | Good |
| State management | useState/useReducer or Zustand | Pinia/Vuex |
| Recommended for | Complex UIs, large teams | Rapid development |

**Recommendation:** Use React for complex UIs, Vue for simpler ones. Both work well.

#### Build Tool: Vite vs Webpack

| | Vite | Webpack |
|---|---|---|
| Config complexity | Low | High |
| Dev server | Fast (ESM) | Slower |
| Bundle size | Smaller | Larger |
| Mature extension support | Good | Excellent |

**Recommendation:** Vite — simpler config, great for extensions.

---

## Step 3: Project Setup

### 3.1 Generate from Template

Based on the user's requirements, generate the appropriate project structure. Create a checklist:

**For single-browser (Chrome) project:**
- [ ] manifest.json with correct permissions
- [ ] background.js service worker
- [ ] popup/ directory with HTML/JS/CSS
- [ ] content/ directory (if needed)
- [ ] icons/ directory with placeholder icons
- [ ] build configuration (vite.config.js)

**For cross-browser project:**
- [ ] Monorepo structure with packages/
- [ ] Base manifest + browser-specific manifests
- [ ] Shared code in common/
- [ ] Browser-specific builds in chrome/, firefox/, etc.
- [ ] Build script to generate each browser's extension

### 3.2 Essential Files Checklist

**manifest.json fields that must be correct:**
- `manifest_version`: Always 3
- `name`: Use `__MSG_keyName__` for i18n
- `version`: Semver format (1.0.0)
- `permissions`: Match actual API usage
- `host_permissions`: Required for MV3 if using `<all_urls>`
- `action.popup`: Points to correct file
- `background.service_worker`: Points to correct file

**Icon requirements per browser:**
| Browser | Sizes Required |
|---------|----------------|
| Chrome | 16, 32, 48, 128 |
| Edge | 16, 32, 48, 128 |
| Firefox | 32, 48, 96, 128 |
| Safari | 48, 96, 128 |

### 3.3 Build Configuration

**vite.config.js for extension with React popup:**
```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: 'extension',
    emptyOutDir: true,
    rollupOptions: {
      input: {
        popup: resolve(__dirname, 'src/popup/index.html'),
        options: resolve(__dirname, 'src/options/index.html'),
        background: resolve(__dirname, 'src/background/index.ts'),
        content: resolve(__dirname, 'src/content/index.ts'),
      },
      output: {
        entryFileNames: '[name]/[name].js',
        chunkFileNames: 'chunks/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash][extname]',
      },
    },
  },
});
```

---

## Step 4: Implementation

### 4.1 Implementation Order

1. **manifest.json** — Define permissions and structure first
2. **background.js** — Core logic and state management
3. **popup UI** — User-facing interface
4. **content scripts** — Page injection (if needed)
5. **options page** — Settings UI (if needed)

### 4.2 Key Patterns

#### Messaging Between Components

```typescript
// content/script.ts
import browser from 'webextension-polyfill';

browser.runtime.sendMessage({
  type: "PAGE_DATA",
  payload: { url: location.href, title: document.title }
});

// background.js
browser.runtime.onMessage.addListener((message, sender) => {
  if (message.type === "PAGE_DATA") {
    // Handle page data
    return Promise.resolve({ received: true });
  }
});
```

#### Storage Pattern

```typescript
// shared/storage.ts
import browser from 'webextension-polyfill';

export interface StorageSchema {
  settings: {
    theme: 'light' | 'dark';
    enabled: boolean;
  };
  cache: Record<string, unknown>;
}

export const storage = {
  async get<K extends keyof StorageSchema>(
    key: K
  ): Promise<StorageSchema[K] | undefined> {
    const result = await browser.storage.local.get(key);
    return result[key];
  },

  async set<K extends keyof StorageSchema>(
    key: K,
    value: StorageSchema[K]
  ): Promise<void> {
    await browser.storage.local.set({ [key]: value });
  },

  onChanged(callback: (changes: Record<string, { oldValue?: unknown; newValue?: unknown }>) => void) {
    browser.storage.onChanged.addListener(callback);
  }
};
```

#### Popup React Component Pattern

```tsx
// popup/App.tsx
import React, { useState, useEffect } from 'react';
import browser from 'webextension-polyfill';
import './popup.css';

interface Bookmark {
  id: string;
  title: string;
  url: string;
  parentId?: string;
}

export function App() {
  const [bookmarks, setBookmarks] = useState<Bookmark[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    browser.bookmarks.getTree().then(tree => {
      // Process bookmarks tree
      setLoading(false);
    });
  }, []);

  if (loading) {
    return <div className="popup-loading">Loading...</div>;
  }

  return (
    <div className="popup-container">
      <h1>Bookmarks</h1>
      {/* Render bookmark tree */}
    </div>
  );
}
```

### 4.3 Error Handling

**Always handle errors gracefully:**

```javascript
// Background script
browser.runtime.onMessage.addListener((message, sender, sendResponse) => {
  handleMessage(message)
    .then(response => sendResponse({ success: true, data: response }))
    .catch(error => {
      console.error('Error:', error);
      sendResponse({ success: false, error: error.message });
    });
  return true; // Keep channel open for async response
});
```

### 4.4 Security Best Practices

- **Minimum permissions** — Only request what you need
- **Host permissions** — Be specific, avoid `<all_urls>` when possible
- **No eval()** — Don't execute strings as code
- **Content script isolation** — Remember content scripts run in isolated world
- **Validate messages** — Check message structure before processing

---

## Step 5: Build & Package

### 5.1 Build Process

```bash
# Install dependencies
npm install

# Build for development (watch mode)
npm run dev

# Build for production
npm run build
```

### 5.2 Load Extension in Browser

**Chrome/Edge:**
1. Go to `chrome://extensions` or `edge://extensions`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select the `extension/` or `chrome/` directory

**Firefox:**
1. Go to `about:debugging#/runtime/this-firefox`
2. Click "Load Temporary Add-on"
3. Select `manifest.json`

**Safari:**
1. Enable Developer menu in Safari → Preferences → Advanced
2. Click "Build Extension" in Developer menu
3. Safari Web Extension appears in Safari preferences

### 5.3 Package for Distribution

**Chrome Web Store:**
```bash
# Create ZIP of extension directory
cd extension && zip -r ../extension.zip . -x "*.map" -x "node_modules/*"
```

**Firefox Add-ons:**
```bash
# Sign via Mozilla Add-ons Developer Hub
# Or use web-ext CLI
npm install -g web-ext
web-ext build
```

**Edge Add-ons:**
- Submit through Microsoft Edge Add-ons dashboard

### 5.4 Build Script Template

```javascript
// build.js - Cross-browser build script
const fs = require('fs-extra');
const path = require('path');

const BROWSERS = ['chrome', 'firefox', 'edge', 'safari'];

async function build() {
  for (const browser of BROWSERS) {
    const srcDir = path.join(__dirname, 'src');
    const outDir = path.join(__dirname, 'packages', browser);

    await fs.ensureDir(outDir);

    // Copy and transform files
    await fs.copy(path.join(srcDir, 'background'), path.join(outDir, 'background'));
    await fs.copy(path.join(srcDir, 'popup'), path.join(outDir, 'popup'));
    await fs.copy(path.join(srcDir, 'content'), path.join(outDir, 'content'));

    // Merge manifests
    const baseManifest = require('./src/manifest/base.json');
    const browserManifest = require(`./src/manifest/${browser}.json`);
    const mergedManifest = { ...baseManifest, ...browserManifest };

    await fs.writeFile(
      path.join(outDir, 'manifest.json'),
      JSON.stringify(mergedManifest, null, 2)
    );

    console.log(`Built ${browser} extension`);
  }
}

build().catch(console.error);
```

---

## Templates

### Minimal Chrome Extension Template

```
extension/
├── manifest.json
├── background.js
├── popup.html
├── popup.js
└── icons/
    ├── icon-48.png
    └── icon-96.png
```

**manifest.json:**
```json
{
  "manifest_version": 3,
  "name": "My Extension",
  "version": "1.0",
  "description": "What it does",
  "permissions": ["storage"],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "48": "icons/icon-48.png"
    }
  },
  "background": {
    "service_worker": "background.js"
  }
}
```

### React Popup Template

```
src/
├── popup/
│   ├── index.html
│   ├── main.tsx
│   ├── App.tsx
│   ├── App.css
│   └── components/
├── background/
│   └── index.ts
├── content/
│   └── index.ts
└── shared/
    ├── storage.ts
    └── types.ts
```

---

## Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Extension not loading | Check manifest.json for errors, use chrome://extensions errors panel |
| Content script not injecting | Check `matches` in manifest, ensure file path is correct |
| Storage quota exceeded | Use `browser.storage.local` for large data, not sync |
| Popup closes immediately | Ensure no JS errors in popup, use browser.runtime API correctly |
| Background script not persisting | MV3 service workers can be terminated; use chrome.storage for state |
| CORS in content script | Content scripts can fetch from same origin only; use background proxy |
| webextension-polyfill not working | Ensure polyfill is loaded before other scripts in manifest |

---

## Next Steps

After creating the extension project structure:

1. **Implement features** following the priority order from discovery
2. **Test on each target browser** before packaging
3. **Use browser devtools** to debug (right-click extension icon → Inspect popup)
4. **Review permissions** before publishing — minimum necessary only

---

## Related Skills

- `tdd-guide` — For writing tests before implementing extension logic
- `code-reviewer` — For reviewing extension code before publishing
- `security-reviewer` — For reviewing permissions and security implications
