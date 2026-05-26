# MindArc Desktop — Project Knowledge Base

> **闪念** (Shǎnniàn) — "Flash of Thought". A local-first, privacy-respecting thought capture desktop app with cyberpunk-inspired bilingual UI.

## Project Overview

**Product**: MindArc Desktop (闪念) — v0.2.2
**Purpose**: Quick-capture tool for fleeting thoughts, ideas, and notes. All data stays local in user-chosen JSON files. No cloud, no accounts, no tracking.
**Tagline**: "心有所念，落笔成篇" (When the mind stirs, let the pen flow.)

## Architecture

### High-Level

```
┌─────────────────────────────────────────────┐
│               index.html (~3400 lines)       │
│  ┌─────────┐ ┌──────────────┐ ┌──────────┐ │
│  │  HTML    │ │  CSS (~1900L)│ │ JS (~1400L)│ │
│  │Template  │ │  Design Sys  │ │  App Logic │ │
│  └─────────┘ └──────────────┘ └──────────┘ │
│           Vanilla JS (IIFE, no framework)    │
└──────────────────┬──────────────────────────┘
                   │ Tauri v2 Plugin APIs (global)
┌──────────────────▼──────────────────────────┐
│            src-tauri/src/main.rs (10 lines)   │
│   tauri-plugin-dialog | tauri-plugin-fs       │
│   tauri-plugin-notification                   │
└──────────────────────────────────────────────┘
```

### Key Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| **Single HTML file** | Zero build step. `index.html` → `dist/` via `cp`. No bundler, no transpiler. |
| **No frontend framework** | App is small enough (~1400 lines of JS). Vanilla DOM manipulation. |
| **Tauri v2 plugins only** | No custom Rust commands. All OS access through official plugins (dialog, fs, notification). |
| **Local JSON file storage** | User picks/creates a `.json` file. Data never leaves the machine. |
| **CSS custom properties** | Design token system for theming (dark/light) and consistent styling. |
| **Bilingual codebase** | UI text in Chinese, code comments bilingual (Chinese / English). |

## File Structure

```
MindArc-Desktop/
├── index.html                  # ENTIRE frontend: HTML + CSS + JS (~3400 lines)
├── package.json                # Minimal: @tauri-apps/cli v2 + 3 plugins
├── package-lock.json
├── .gitignore                  # node_modules, dist, src-tauri/target
├── AGENTS.md                   # This file
│
├── src-tauri/                  # Rust / Tauri backend
│   ├── Cargo.toml              # tauri v2, dialog/fs/notification plugins, serde
│   ├── build.rs                # tauri_build::build()
│   ├── tauri.conf.json         # App config: window 960x720, min 600x500
│   ├── src/
│   │   └── main.rs             # Plugin registration only (10 lines)
│   ├── capabilities/
│   │   └── default.json        # Permissions: dialog, fs (home+app r/w), notification
│   ├── icons/                  # App icons (32x32, 128x128, icns, ico)
│   └── gen/                    # Auto-generated Tauri scaffolding
│
├── .github/workflows/
│   ├── build.yml               # CI: build macOS (arm64+x64) + Windows on push to main
│   └── release.yml             # CD: publish releases on v* tags
│
└── dist/                       # Build output (gitignored, created by cp index.html)
```

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Desktop Shell | **Tauri** | v2 |
| Backend | **Rust** | 2021 edition |
| Frontend | **Vanilla JavaScript** | No framework, IIFE pattern |
| Styling | **CSS Custom Properties** | No preprocessor |
| Data Format | **JSON** | serde_json v1 |
| Build | **npm scripts** → `tauri build` | No Webpack/Vite |
| CI/CD | **GitHub Actions** | Node 22 + Rust stable |

### Tauri Plugins Used

| Plugin | Purpose |
|--------|---------|
| `tauri-plugin-dialog` | File open/save dialogs |
| `tauri-plugin-fs` | Read/write local JSON files |
| `tauri-plugin-notification` | System notifications for reminders |

## Data Model

```json
{
  "version": 1,
  "thoughts": [
    {
      "id": "crypto.randomUUID()",
      "content": "The thought text",
      "createdAt": "ISO 8601 datetime",
      "completedAt": "ISO 8601 datetime | null",
      "priority": "q1 | q2 | q3 | q4",
      "tag": "string (e.g. Team, AI, Coding, Mine, or custom)",
      "reminder": {
        "enabled": false,
        "datetime": "ISO 8601 datetime | null",
        "repeat": "none | daily | weekly | monthly"
      }
    }
  ],
  "customTags": ["string"]
}
```

### Priority System (Eisenhower Matrix)

| Key | Label | Color | Meaning |
|-----|-------|-------|---------|
| `q1` | 急要 (Urgent & Important) | `#ff4081` pink | Do first |
| `q2` | 重要 (Important) | `#00e5ff` cyan | Schedule |
| `q3` | 紧急 (Urgent) | `#ffab40` orange | Delegate |
| `q4` | 闲记 (Leisure) | `#7a80a8` muted | Later |

### Default Tags

`Team`, `AI`, `Coding`, `Mine` — users can add custom tags.

## Frontend Architecture (index.html)

The entire frontend lives in one file structured as:

```
index.html
├── <style>         (~1900 lines)  CSS design system + all component styles
├── <body>          HTML templates
│   ├── #welcomeScreen     Welcome/create-or-open-file overlay
│   ├── #mainApp           Main thought list + header filters
│   └── #bottomBar         Fixed bottom input bar (priority + tags + textarea)
└── <script>        (~1400 lines)  Application logic in IIFE
```

### JavaScript Module Structure (within IIFE)

| Section | Lines | Responsibility |
|---------|-------|---------------|
| State Management | ~20 | Module-level variables (thoughts, filters, tags, etc.) |
| Reminder Engine | ~200 | Notification permission, check loop, repeat scheduling, editor UI |
| Popup Manager | ~80 | Reusable floating popup component |
| DOM Refs | ~15 | Cached getElementById references |
| Init | ~15 | Load saved file path from localStorage |
| File Operations | ~80 | createNewFile, openExistingFile, loadFromFile, saveData |
| UI Switching | ~15 | Show/hide welcome vs main app |
| Thought CRUD | ~70 | addThought, toggleThought, deleteThought, persistAndRender |
| Rendering | ~300 | render(), createCardElement(), renderTagView() |
| Filtering | ~30 | getFilteredThoughts() — status + priority + tag + search |
| Tag Helpers | ~100 | getAllTags, renderTagSelector, renderTagFilter, setupAddTagInline |
| Time Formatting | ~40 | formatRelativeTime, formatAbsoluteTime |
| Event Binding | ~100 | All DOM event listeners |
| Reminder UI | ~170 | Panel setup, toggle, live state updates |
| Font Size | ~15 | A-/A+ buttons, localStorage persistence |
| Theme Toggle | ~30 | Dark/light, system preference detection, localStorage |

### State Variables

```javascript
let filePath = null;              // Current JSON file path
let thoughts = [];                // All thought objects
let currentFilter = 'all';        // 'all' | 'pending' | 'completed'
let searchQuery = '';             // Text search
let currentPriority = 'q1';      // Input priority for new thoughts
let currentPriorityFilter = 'all'; // Display filter
let currentTag = 'Mine';           // Input tag for new thoughts
let currentTagFilter = 'all';      // Display filter
let customTags = [];               // User-created tags
let currentView = 'list';          // 'list' | 'tags'
let currentReminder = { enabled, datetime, repeat };
```

### CSS Design System

- **CSS Custom Properties** defined in `:root` and `[data-theme="light"]`
- **Color palette**: Cyberpunk-inspired — cyan (#00e5ff), blue (#448aff), purple (#b388ff), pink (#ea80fc)
- **Dark mode**: Default (`#0a0a1a` background)
- **Light mode**: `[data-theme="light"]` overrides
- **Spacing**: 12px border-radius, 250ms cubic-bezier transitions
- **Responsive**: Mobile-first with `@media (max-width: 480px)` and `(min-width: 768px)`
- **Accessibility**: `focus-visible` outlines, `prefers-reduced-motion` support, ARIA attributes

## Build & Development

### Commands

```bash
npm install          # Install Tauri CLI + plugins
npm run dev          # Start dev server with hot reload (tauri dev)
npm run build        # Production build (tauri build)
npm run tauri        # Direct tauri CLI access
```

### Build Pipeline

1. `npm install` — install Node dependencies
2. `mkdir -p dist && cp index.html dist/index.html` — copy frontend (no compilation!)
3. `tauri build` — compiles Rust, bundles with frontend into platform installer

### Output Artifacts

| Platform | Artifact |
|----------|----------|
| macOS ARM64 | `.dmg`, `.app.tar.gz` |
| macOS x64 | `.dmg`, `.app.tar.gz` |
| Windows | `.msi`, `.exe` (NSIS) |

## CI/CD

- **Build** (`.github/workflows/build.yml`): Triggered on push to `main`. Builds macOS (arm64 + x64) and Windows. Uploads artifacts.
- **Release** (`.github/workflows/release.yml`): Triggered on `v*` tags. Same matrix build, publishes to GitHub Releases.

## Key Patterns & Conventions

### Code Style

- **JavaScript**: IIFE strict mode, no modules, no async/await at top level
- **Tauri API access**: Via `window.__TAURI__` global (configured in `tauri.conf.json` → `withGlobalTauri: true`)
- **DOM**: Imperative createElement + textContent (XSS-safe), no innerHTML for user data
- **Naming**: `$variableName` for DOM element references (jQuery convention)
- **State persistence**: `localStorage` for UI preferences (theme, font size, file path); JSON file for data
- **Comments**: Bilingual — section headers like `// ========== 标签名 / English Name ==========`

### UI Patterns

- **Fixed bottom bar**: Input always visible, frosted glass backdrop-filter
- **Card list**: Each thought is an `<article>` card with left color stripe by tag
- **Inline editing**: Click edit → textarea replaces text, with save/cancel
- **Inline delete confirmation**: "当真?" (Are you sure?) with "然/且慢" (Yes/Wait)
- **Popup menus**: Custom floating `<div>` for priority/tag pickers, positioned relative to anchor
- **Animations**: CSS `@keyframes` for card appear/remove, edit pulse, reminder glow

### Error Handling

- `try/catch` around all Tauri API calls with `console.warn`/`console.error`
- Graceful fallback: if file read fails, show empty state
- Defensive checks: `if (!window.__TAURI__)` for notification API

## Important Notes for Contributors

1. **No build step for frontend**: Edit `index.html` directly. No compilation, no hot module replacement in the traditional sense.
2. **Rust backend is minimal**: All logic is in the frontend JS. The Rust side only registers plugins. To add native functionality, add a Tauri plugin or write custom commands in `main.rs`.
3. **Data format stability**: The JSON schema uses `version: 1`. Any schema changes must maintain backward compatibility (see `loadFromFile` which adds missing `reminder` fields).
4. **No tests exist**: No test files or test runners configured. Adding tests would require setting up a test framework.
5. **Chinese-first UI**: All user-facing text is in Chinese with classical/literary style (e.g., "落笔新卷", "翻阅旧卷", "抹去此念"). Maintain this tone.
6. **Privacy by design**: No network calls, no analytics, no telemetry. All data is in user's local file. Keep it that way.
