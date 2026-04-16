---
name: browser-extension-creator
description: |
  Create and maintain cross-browser extensions (Chrome, Edge, Firefox, Safari). Use when the user wants to build a new browser extension, modify an existing extension, or get guidance on extension development.

  Triggers: "浏览器插件", "browser extension", "Chrome扩展", "帮我做个插件", "extension manifest", "browser extension project structure"
---

# Browser Extension Creator

Create cross-browser extensions with a structured 5-phase workflow.

## NON-NEGOTIABLE Constraints

- **Manifest V3 only** — All browsers now support MV3. Never use V2.
- **Minimum permissions** — Only request what you actually need. Fewer permissions = more trust.
- **Service worker lifecycle** — Background scripts terminate when idle. Never rely on in-memory state.
- **Use `browser.*` namespace** — Firefox/Safari standard. Add `webextension-polyfill` for Chrome compatibility.
- **No remote code** — All JS must be bundled locally in MV3.

---

## Phase 0: Detect Intent

Determine what the user needs:

- **Mode A: New Extension** — Build from scratch → Go to Phase 1
- **Mode B: Modify Existing** — User has existing extension code → Ask to see it
- **Mode C: Question Only** — User has a specific question → Answer directly

### Mode B: Modifying Existing

Ask the user to share their current extension files. Read and understand:
1. manifest.json — Check MV2 vs MV3, permissions, structure
2. Background script — Identify persistence patterns that need changing
3. Content/popup scripts — Check for deprecated APIs

Provide targeted guidance based on what you find.

---

## Phase 1: Discovery

**Ask ALL questions in one batch** (single AskUserQuestion call):

**Question 1 — Purpose** (header: "Extension Type"):
What does this extension do? Options: Bookmark tool / Tab manager / Content helper / Productivity tool / Other

**Question 2 — Browser Target** (header: "Browsers"):
Which browsers need support? Options: Chrome only / Chrome + Edge / All major (Chrome, Edge, Firefox, Safari)

**Question 3 — Key Features** (header: "Features"):
What are the core features? Select all that apply: Popup UI / Options page / Context menu / Content scripts / Background processing / Data storage

**Question 4 — Permissions** (header: "Permissions"):
Does it need special browser access? Options: Basic (tabs, storage only) / Bookmarks / Downloads / Active tab / Custom sites / Not sure

If user selects "Not sure" for permissions, ask them to describe what the extension needs to access or do.

---

## Phase 2: Architecture

Based on Phase 1 answers, determine:

### Project Structure

| Single Browser (Chrome) | Cross-Browser |
|------------------------|---------------|
| Flat structure with single manifest.json | Monorepo with `packages/chrome/`, `packages/firefox/` |
| Faster to build | Shared code in `src/common/` |

### Manifest Strategy

```json
// Cross-browser: manifest layering
src/manifest/
├── base.json      // Shared config (MV3, permissions, action)
├── chrome.json    // Chrome-specific overrides only
└── firefox.json   // Firefox-specific overrides only
```

### Browser Support Matrix

| Feature | Chrome | Edge | Firefox | Safari |
|---------|--------|------|---------|--------|
| Manifest V3 | ✅ | ✅ | ✅ | ✅ |
| Service Worker | ✅ | ✅ | ✅ | ✅ |
| browser.* API | +polyfill | +polyfill | ✅ | ✅ |
| Bookmarks API | ✅ | ✅ | ✅ | ✅ |

---

## Phase 3: Project Setup

Generate the project structure based on decisions in Phase 2.

### Output Structure

```
extension/
├── manifest.json          # Or src/manifest/ for cross-browser
├── background.js         # Service worker
├── content/
│   └── script.js         # Content script (if needed)
├── popup/
│   ├── popup.html
│   └── popup.js
├── options/
│   └── options.html      # Options page (if needed)
├── icons/
│   ├── icon-48.png
│   └── icon-96.png
└── _locales/
    └── en/messages.json  # i18n (if needed)
```

### Build Configuration

```javascript
// vite.config.js for React/Vue popup
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: 'extension',
    rollupOptions: {
      input: {
        popup: 'src/popup/index.html',
        background: 'src/background/index.ts',
      }
    }
  }
});
```

---

## Phase 4: Implementation

### Manifest Template

```json
{
  "manifest_version": 3,
  "name": "__MSG_extName__",
  "version": "1.0.0",
  "description": "__MSG_extDescription__",
  "permissions": ["storage"],
  "host_permissions": [],
  "background": {
    "service_worker": "background.js",
    "type": "module"
  },
  "action": {
    "default_popup": "popup/popup.html"
  }
}
```

### Key Patterns

**Messaging:**
```javascript
// content script → background
browser.runtime.sendMessage({ type: "GET_DATA", url: location.href })
  .then(response => console.log(response));

// background → content
browser.tabs.query({ active: true }).then(([tab]) => {
  browser.tabs.sendMessage(tab.id, { type: "UPDATE", data });
});
```

**Storage:**
```javascript
browser.storage.local.set({ key: value });
browser.storage.local.get("key").then(result => /* use result */);
```

**Service Worker Lifecycle:**
```javascript
// Cache state — service worker terminates between events
chrome.runtime.onInstalled.addListener(() => {
  chrome.storage.local.set({ cachedData: "persist here" });
});

// For periodic tasks, use alarms
chrome.alarms.create("periodicTask", { periodInMinutes: 5 });
chrome.alarms.onAlarm.addListener(alarm => {
  if (alarm.name === "periodicTask") doWork();
});
```

---

## Phase 5: Build & Test

### Load Extension in Browser

| Browser | Steps |
|---------|-------|
| Chrome/Edge | `chrome://extensions/` → Developer mode → Load unpacked → Select folder |
| Firefox | `about:debugging#/runtime/this-firefox` → Load Temporary Add-on |
| Safari | Safari → Developer menu → Build Extension |

### Package for Store

```bash
# Chrome Web Store
zip -r extension.zip extension/ -x "*.map"

# Firefox
npx web-ext build
```

---

## Common Issues

| Issue | Solution |
|-------|----------|
| Extension not loading | Check `chrome://extensions/` errors panel |
| Background script errors | Service worker terminated? Check `chrome.storage` for state |
| Content script not injecting | Verify `matches` pattern in manifest |
| Storage quota exceeded | Use `storage.local` (~10MB), not `storage.sync` (~100KB) |
| CORS in content script | Use background script as proxy |

---

## Supporting Files

| File | Purpose | When to Read |
|------|---------|--------------|
| `evals/evals.json` | Test cases for skill validation | Skill development |
