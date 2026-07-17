# 🍅 PomoPal

**Focus with a friendly rhythm.**

PomoPal is a pomodoro-based study companion built as one HTML file (plus a small `apple-touch-icon.png` for iOS home-screen installs) — no build step, no dependencies, no backend, no account. Open `index.html` in a browser and start focusing. Everything saves locally to your browser, and a skippable guide walks you through it on first visit.

## Features

### ⏱ Timer
- Classic pomodoro: 25 min focus / 5 min short break / 15 min long break after four rounds (all adjustable with stepper controls)
- A gradient SVG progress ring with a breathing glow — coral for focus, sage for breaks — and a gentle "cheer" when a session completes
- Three tiny icons under the ring (brain / coffee cup / couch) show today's completed focus sessions, short breaks, and long breaks, and jump straight to any phase on tap — asks to confirm first if a session is actively running
- **Working on** sits right under the clock as a compact subject pill; tap it to pick from today's planned tasks — finished sessions log themselves and tick off that task's pomodoro count
- Survives page reloads mid-session, counts down in the tab title, catches up instantly when a background tab wakes
- Chime (three styles + volume), desktop notification when a phase ends in a background tab
- Auto-continue mode, `Space` to start/pause (while the Timer tab is showing and the guide isn't open), and a fullscreen mode (top-right) that hides the cursor while you work
- Settings, ambience, and focus music live in a collapsible sheet pinned to the bottom of the screen

### 📋 Plan
Two views behind a segmented control — **Today** for the day's list, **Targets** for longer-range aims.
- **Today:** plan sessions per subject with a pomodoro estimate; drag to reorder (↑/↓ buttons on touch), check off with custom circular checkboxes, edit notes in place; on touch, swipe a task right to complete it or left for Edit note / Delete. Tasks auto-complete when their pomodoro count is reached.
- **Targets:** aim at something bigger than today. **Pomodoro** and **Minute** targets fill up on their own — from live focus sessions and logged study time — with an optional subject scope; **Milestones** (manual) tick with + and −. Add an optional deadline with days-left display, and progress bars pulse when a target is freshly completed.

### 📖 Session log
- Every finished focus session lands here on its own; log longer sessions by hand with notes
- **Streak counter** and a **7-day bar chart**, plus stat cards for today, the last 7 days, total sessions, and streak
- Grouped by day (Today, Yesterday, …), searchable, filterable by color-coded subject chips
- Every delete shows an **Undo** toast — nothing is lost to a mis-tap

### 🎵 Focus music & ambience
- Built-in ambient **rain** and **deep noise**, generated with WebAudio — no audio files
- YouTube search with an audio-only player: track art, play/pause, seek, prev/next, volume, live-stream aware
- Preset stations: lofi, synthwave, jazz, classical, rain — or paste any YouTube link/playlist

### ✨ Everything else
- Warm light theme and a deep-plum dark theme (◐ in the header), custom-designed fields, buttons, and segmented navigation
- Reusable color-coded subjects shared across the timer, plan, and log — created through a small in-app dialog, not the browser's prompt
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
- Most icons are inline SVGs with Material geometry (no icon font); a handful of controls use plain text glyphs (◐, ✕, ↑↓, ✦) chosen to be emoji-safe on iOS.

## Known platform limits

- Chimes can't sound while a phone is locked (the timer itself catches up correctly).
- YouTube/ambience audio pauses in background mobile tabs (browser policy).
- Desktop notifications require permission; on Android they'd need a service worker, so they're skipped gracefully there.
- iPhone Safari has no Fullscreen API for arbitrary elements — zen mode still applies its layout there, just without hiding the browser chrome.

## License

MIT
