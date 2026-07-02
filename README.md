# 🍅 PomoPal

**Focus with a friendly rhythm.**

PomoPal is a pomodoro-based study companion in a single HTML file — no build step, no dependencies, no backend, no account. Open `index.html` in a browser and start focusing. Everything saves locally to your browser, and a skippable guide walks you through it on first visit.

## Features

### ⏱ Timer
- Classic pomodoro: 25 min focus / 5 min short break / 15 min long break after four rounds (all adjustable with stepper controls)
- A gradient SVG progress ring with a breathing glow — coral for focus, sage for breaks — and a gentle "cheer" when a session completes
- Survives page reloads mid-session, counts down in the tab title, catches up instantly when a background tab wakes
- Chime (three styles + volume), desktop notification when a phase ends in a background tab
- Auto-continue mode, `Space` to start/pause, and a fullscreen mode (top-right) that hides the cursor while you work
- Link a planned task with **Working on** — finished sessions log themselves

### 📋 Planner
- Plan sessions per subject with a pomodoro estimate
- Drag to reorder with a live insertion indicator (↑/↓ buttons on touch), check off with custom circular checkboxes, edit notes in place
- Tasks auto-complete when their pomodoro count is reached

### 🎯 Goals
- **Count goals** tick with + and −
- **Minute goals** track your logged study time automatically — scope them to a subject, add an optional deadline with days-left display
- Progress bars pulse when a goal is freshly completed

### 📖 Session log
- Every finished focus session lands here on its own; log longer sessions by hand with notes
- **Streak counter** and a **7-day bar chart**, plus today / week / total stat cards
- Grouped by day (Today, Yesterday, …), searchable, filterable by color-coded subject chips
- Every delete shows an **Undo** toast — nothing is lost to a mis-tap

### 🎵 Focus music & ambience
- Built-in ambient **rain** and **deep noise**, generated with WebAudio — no audio files
- YouTube search with an audio-only player: track art, play/pause, seek, prev/next, volume, live-stream aware
- Preset stations: lofi, synthwave, jazz, classical, rain — or paste any YouTube link/playlist

### ✨ Everything else
- Warm light theme and a deep-plum dark theme (◐ in the header), custom-designed fields, buttons, and segmented navigation
- Reusable color-coded subjects shared across planner, log, and goals
- One-click JSON backup export/import (bottom of the Session log tab)
- Installable as a web app (Android/Chrome manifest + iOS metas)
- Reduced-motion support and WCAG AA contrast in both themes

## Getting started

```
git clone https://github.com/figno-ai-radiologist/Pomopal.git
```

Open `index.html` in any modern browser. That's it.

Your data lives in `localStorage` under `study.*` keys — export a backup from the Session log tab before switching browsers or machines.

## Tech notes

- **One file.** HTML + CSS + vanilla JS, zero dependencies, zero build.
- **Brand tokens** live in the CSS variable block at the top of `index.html` — a rebrand is a one-block edit.
- Music search uses public Piped/Invidious instances (keyless, CORS-open) with a fallback chain; playback uses a hidden YouTube iframe driven via the postMessage API. Public instances rotate — if search dies, refresh the instance list near `MUSIC_APIS`.
- The timer computes from wall-clock timestamps, so it stays accurate through tab throttling and reloads.
- Icons are inline SVGs with Material geometry — no icon font, nothing for iOS to render as emoji.

## Known platform limits

- Chimes can't sound while a phone is locked (the timer itself catches up correctly).
- YouTube/ambience audio pauses in background mobile tabs (browser policy).
- Desktop notifications require permission; on Android they'd need a service worker, so they're skipped gracefully there.

## License

MIT
