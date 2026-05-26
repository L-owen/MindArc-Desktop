# 闪念 MindArc

> 心有所念，落笔成篇

A local-first, privacy-respecting thought capture desktop app. All data stays on your machine — no cloud, no accounts, no tracking.

## Features

- **Quick Capture** — One input bar, always visible at the bottom. Jot down any thought in seconds.
- **Eisenhower Matrix** — Prioritize thoughts with four levels: Urgent & Important, Important, Urgent, Leisure.
- **Tags** — Organize by context (Team, AI, Coding, Mine) or create your own.
- **Auto File Memory** — Remembers your last opened file. Pick up right where you left off.
- **Reminders** — Set one-time or repeating reminders with system notifications.
- **Auto Backup** — Creates `.bak` backup before every save. Your data is safe.
- **Dark / Light Theme** — Cyberpunk-inspired dark mode by default, with light mode toggle.
- **Cross-Platform** — macOS (Apple Silicon + Intel) and Windows.

## Privacy

- All data in a single local `.json` file you choose
- No network calls, no analytics, no telemetry
- File stored wherever you want — Documents, desktop, external drive

## Screenshots

![alt text](image.png)

## Download

Latest release: [v0.3.0](https://github.com/L-owen/MindArc-Desktop/releases/tag/v0.3.0)

| Platform | Download |
|----------|----------|
| macOS (Apple Silicon) | `.dmg` |
| macOS (Intel) | `.dmg` |
| Windows | `.exe` or `.msi` |

## Development

```bash
npm install          # Install dependencies
npm run dev          # Start dev server with hot reload
npm run build        # Production build
```

**Tech stack:** Tauri v2 + Vanilla JavaScript + CSS Custom Properties

No build step for the frontend — edit `index.html` directly.

## License

MIT
