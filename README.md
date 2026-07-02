# 🍅 PomoPal

**Focus with a friendly rhythm.**

PomoPal is a pomodoro-based study companion in a single HTML file — no build step, no dependencies, no backend, no account. Open `index.html` in a browser and start focusing. Everything saves locally to your browser.

## Features

### ⏱ Timer
- Classic pomodoro: 25 min focus / 5 min short break / 15 min long break after four rounds (all adjustable)
- Progress ring with a soft breathing glow — coral for focus, sage for breaks
- Survives page reloads mid-session, counts down in the tab title
- Chime (three styles + volume), desktop notification when a phase ends in a background tab
- Auto-continue mode, `Space` to start/pause, and a fullscreen **Zen** mode
- Link a planned task with **Working on** — finished sessions log themselves

### 📋 Planner
- Plan sessions per subject with a pomodoro estimate
- Drag to reorder, check off as you go, edit notes in place
- Tasks auto-complete when their pomodoro count is reached

### 🎯 Goals
- **Count goals** tick with + and −
- **Minute goals** track your logged study time automatically — scope them to a subject, add an optional deadline with days-left display

### 📖 History
- Every finished focus session lands here on its own; log longer sessions by hand with notes
- Search, filter by color-coded subject chips, edit notes in place
- Today / last-7-days minute totals at a glance

### 🎵 Focus music & ambience
- Built-in ambient **rain** and **deep noise**, generated with WebAudio — no audio files
- YouTube search with a proper audio-only player: track list, play/pause, seek, prev/next, volume, live-stream aware
- Preset stations: lofi, synthwave, jazz, classical, rain — or paste any YouTube link/playlist

### ✨ Everything else
- Warm light theme and a deep-plum dark theme (◐ in the header)
- Reusable color-coded subjects shared across planner, history, and goals
- One-click JSON backup export/import (⤓ / ⤒)
- A skippable first-run guide (reopen with **?**)
- Reduced-motion support and WCAG AA contrast in both themes

## Getting started

```
git clone https://github.com/figno-ai-radiologist/Pomopal.git
```

Open `index.html` in any modern browser. That's it.

Your data lives in `localStorage` under `study.*` keys — export a backup from the header before switching browsers or machines.

## Tech notes

- **One file.** HTML + CSS + vanilla JS, zero dependencies, zero build.
- **Brand tokens** live in the CSS variable block at the top of `index.html` — a rebrand is a one-block edit.
- Music search uses public Piped/Invidious instances (keyless, CORS-open) with a fallback chain; playback uses a hidden YouTube iframe driven via the postMessage API. Public instances rotate — if search dies, refresh the instance list near `MUSIC_APIS`.
- The timer computes from wall-clock timestamps, so it stays accurate through tab throttling and reloads.

## License

MIT
