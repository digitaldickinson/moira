# Moira — Pending Work

## Mobile / Tablet (remaining)

These were identified during the v2.7.0 mobile audit but deferred.

- [ ] **TX / Director bar on mobile** — the live ON AIR bar (timer, backtime, Take Next, cue controls) is a single non-wrapping flex row. Needs `flex-wrap` and responsive spacing so it's usable when operating TX from a tablet. `renderDirector` ~line 3153.

- [ ] **Presenter alert banner wrapping** — the sticky alert bar shown during TX has alert text and dismiss button in a non-wrapping row; long alerts truncate on narrow screens. Should wrap to `flex-col sm:flex-row`. ~line 4037.

- [ ] **Radio playout deck UI** — deck controls are desktop-only in practice (Studio PC), but worth a tidy-up pass for completeness. Lower priority.

- [ ] **Autocue touch-scroll** — verify momentum scrolling works smoothly on iPad. Likely fine (`-webkit-overflow-scrolling: touch` is set on `#main-container`) but not yet tested on device.

---

## Features / Ideas (backlog)

*(Add future feature ideas here)*
