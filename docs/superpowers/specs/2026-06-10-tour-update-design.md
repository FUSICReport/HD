# Onboarding Tour Update

**Date:** 2026-06-10
**Status:** Approved

## Problem

The tour predates the burger-menu redesign and the workflow-view work:

- Step 1 targets `.header-actions`, which no longer exists — its spotlight is
  silently broken and its text references header icons that are gone.
- The tour never mentions the Report View / Workflow View choice.
- The export step is a single sentence; it does not cover the export modal or
  the image-inclusion feature.

## Design

### Mechanics

- Expose the drawer's `openDrawer` as `window._openNavDrawer` (mirroring the
  existing `window._closeNavDrawer`) so tour `before()`/`after()` hooks can
  open/close the burger menu — same pattern the Settings step uses.
- Steps gain optional `noScroll: true` and `delay` (ms) properties.
  `_goToTourStep` reads them instead of the hard-coded selector list (which
  still names the dead `.header-actions`). Settings step gets `delay: 400`;
  the export-modal step too.
- The tour overlay (z 1100+) already sits above the drawer (200) and modals
  (300), so no z-index work is needed.

### Steps (9 total)

1. **Welcome** — retargeted to `#burgerBtn` (round spotlight, noScroll); text
   rewritten to point at the menu button.
2. **NEW Report vs Workflow** — `before()` opens the drawer, spotlights
   `.nav-view-toggle` (noScroll); text: Report View is ordered like the final
   report (write-up afterwards); Workflow View has the identical fields
   organised by echo window (PLAX, PSAX, A4C…) for entry while scanning; both
   share the same data, switch any time. `after()` closes the drawer.
3. Scan information — unchanged.
4. Form sections — unchanged.
5. FUSIC HD questions — unchanged.
6. Clinical conclusions — unchanged.
7. Settings — unchanged (gains `delay: 400` via the new property).
8. **Exporting** — `#stickyFooter` (noScroll), text tightened to introduce
   the Export button.
9. **NEW Export options** — `before()` calls `openExportModal()`, spotlights
   `.modal-box--export` (noScroll, delay 400); text covers the formats
   (plain text, print/PDF, Excel, BSE/EDEC), the "Include images in report"
   toggle (PDF-only embedding, select/reorder/review before export), and
   pseudo-anonymisation. `after()` calls `closeExportModal()`.

## Verification

Playwright:

1. Walk all 9 steps forward; every step's target exists and the spotlight is
   positioned (non-zero size, opacity 1).
2. Drawer opens on step 2 and closes on step 3; export modal opens on step 9
   and closes on Finish.
3. Walk backwards across the drawer step — drawer reopens and closes
   correctly.
4. Skip mid-step-2 and mid-step-9 — drawer/modal are closed by `endTour()`.
