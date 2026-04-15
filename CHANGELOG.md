# Changelog

All notable changes to Moira will be documented in this file.

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
