# Moira — Offline Teleprompter

<img width="910" height="612" alt="image" src="https://github.com/user-attachments/assets/c81f7b51-2821-42a5-80f6-2bb7361d87e2" />



A single-file, offline prompter built for broadcast emergencies. No server, no internet connection, no installation. Open in Microsoft Edge and go.

Try it at https://digitaldickinson.github.io/moira/OfflinePrompt/moira-prompt.html 

---

## Quick Start

1. Save `moira-Prompt.html` anywhere on the machine
2. Open it in Edge
3. Load a `.txt` script file (or paste text directly)
4. Press **Space** or click **▶ SCROLL** to start

---

## Two-Screen Setup (Controller + Presenter)

The app supports a mirrored presenter view synced to the operator's controller view in real time — no server required.

1. Open `moira-Prompt.html` in Edge on the **operator screen**
2. Load your script
3. Click **Open Presenter Tab** — this opens a second tab with `#presenter` in the URL
4. Drag that tab to the **presenter monitor** and go fullscreen (`F`)

The presenter tab auto-mirrors horizontally on open (ready for a teleprompter glass). Scroll position, speed, font size, and reading line state all sync instantly between windows. The two screens **do not need to be the same size** — sync is proportional, not pixel-based.

> **Important:** Both tabs must open the same saved copy of `moira-Prompt.html`. If you move the file, reopen both tabs.

---

## Script Format

Plain `.txt` files. The app also accepts pasted text directly on the load screen.

| Syntax | Effect |
|---|---|
| `## Section Name` | Yellow section header pill |
| `[CUE MUSIC]` | Orange cue tag pill |
| `[CLIP:0:45]` | Amber clip timer pill |
| `*word*` | Bold emphasis |
| Blank line | Paragraph break |

---

<img width="1249" height="1024" alt="image" src="https://github.com/user-attachments/assets/3c9b0124-226a-4f07-8c09-384438530da6" />


## Controls

### Keyboard (Operator)

| Key | Action |
|---|---|
| `Space` | Start / pause scroll |
| `L` | Speed up (and start if paused) |
| `J` | Slow down (stops at minimum speed) |
| `K` | Stop |
| `↓` / `↑` | Jump to next / previous `##` section |
| `+` / `−` | Increase / decrease font size |
| `[` / `]` | Decrease / increase scroll speed |
| `Home` / `End` | Jump to top / bottom |
| `F` | Toggle fullscreen |
| `M` | Toggle horizontal mirror |
| `R` | Toggle reading line |

### Keyboard (Presenter tab)

| Key | Action |
|---|---|
| `M` | Toggle horizontal mirror |
| `R` | Toggle reading line |
| `F` | Toggle fullscreen |

### Controls Bar

Auto-hides after 3 seconds of inactivity. Move the mouse or press any key to restore it.

| Control | Location |
|---|---|
| Font size − / + | Left |
| Mirror X / Y | Centre |
| Reading line toggle | Centre |
| ▶ SCROLL / ⏸ PAUSE | Centre |
| Open Presenter Tab | Centre (operator only) |
| Load new file | Centre (operator only) |
| Fullscreen | Centre |
| Speed − / + | Right (operator only) |

---

## Features

**Reading line** — a fixed blue band at 38% down the screen marks the natural eye resting position. The script scrolls behind it. Toggle with `R` or the bar button. Syncs between tabs.

**Time remaining** — top-left corner shows `M:SS` based on current position and speed. Goes amber under 60 seconds, red under 30.

**Clock** — live `HH:MM:SS` displayed in the top-right of the presenter tab.

**Progress bar** — thin blue line at the very top of the screen.

**Sync** — uses `localStorage` storage events to communicate between tabs. Position is synced as a proportion of total script length (not pixels) so windows of different sizes stay perfectly in step. Speed changes use a dead-reckoning approach — no frame-by-frame polling, no drift.

**Paste mode** — the load screen has a paste textarea for copying script directly from email or a shared document without saving a file first.

**Clear saved script** — a link at the bottom of the load screen removes any script stored in browser memory, so stale copy doesn't auto-load next session.

---

## Limitations

- Requires Microsoft Edge (or any Chromium browser) — Chrome, Edge, and Brave all work
- Two-tab sync only works when both tabs open the **exact same file path** (same `file://` origin)
- Script must be plain text — no Word or PDF import
- No network features; fully offline by design

---

*Part of the Moira broadcast tools suite.*
