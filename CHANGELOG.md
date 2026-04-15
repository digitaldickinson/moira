# Changelog

All notable changes to Moira will be documented in this file.

## [2.7.0] - 2026-04-15
### Added
- **Mobile & tablet layout**: Moira is now usable on iPad and mobile browsers.
  - Header collapses on small screens: logo text, staff button, and room input hide below `md`; tab labels shrink to icon-only on mobile.
  - Settings modal scrolls correctly on iPad (fixed `min-h-0` flexbox overflow).
  - Presenter cards: font-size toggle button (`fa-text-height`) scales script text between normal and large for easier reading on mobile.
  - Autocue control buttons enlarged to 44 × 44 px minimum tap targets; iOS safe-area inset applied to the fixed bottom bar.
  - Rundown: toolbar and timing bar wrap on narrow screens; desktop table hidden below `lg` breakpoint in favour of a card-based layout with Edit / Check / Float / ▲ / ▼ actions per story.
  - Touch drag-and-drop on iPad via `mobile-drag-drop` polyfill (hold 400 ms to initiate); ▲ / ▼ buttons remain as offline fallback.
  - Story edit modal form fields stack to single column on mobile; script textarea no longer clips on portrait phones.
  - Alert popover and toast/error notifications constrained to viewport width on narrow screens.

### Fixed
- **Studio radio deck buttons always triggered Deck A**: Play, Pause, Stop, Cue, and scrub bar used `data-deck` but the click handler read `data-id`, so Deck B controls silently operated Deck A instead.

---

## [2.6.0] - 2026-04-15
### Added
- **Presenter clip remote handshake**: During a live TX, clip buttons in the presenter view now always route audio through the Studio PC (the machine with the media folder open), regardless of which device fires the request.
- **Rehearsal mode**: Outside of TX, any machine with the media folder connected plays clips locally. Students and journalists can run through their bulletin and hear clips on their own machine without involving the Studio PC.
- **Audio Master badge**: The TX status bar shows a green `AUDIO MASTER` indicator on whichever machine is acting as the audio output for presenter clips.
- **Clip buttons on all clients**: Presenter clip fire buttons now appear on all connected machines during TX, not just the machine with the media folder.

### Fixed
- Button PLAY/STOP state now reflects what the master is playing across all clients, not just the local machine.
- Tapping a playing clip's button on any client (including non-master) now correctly stops playback on the master.
- Switching clips while one is playing no longer causes both to play simultaneously.

---

## [2.5.0] - prior
### Notes
- Stable management build. No changelog recorded prior to v2.6.0.
