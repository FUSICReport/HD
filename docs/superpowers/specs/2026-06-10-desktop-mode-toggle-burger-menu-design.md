# Desktop Mode Toggle in Burger Menu

**Date:** 2026-06-10
**Status:** Approved

## Problem

The "Desktop mode" (compact mode) toggle lives in the Settings panel's Display
group, where it is hard to discover. It should sit in the burger-menu drawer
directly below "Dark mode" — but only on screens wide enough for the mode to
matter (PCs), and be completely absent on phones.

## Design

1. **New drawer row** below "Dark mode", using the same `nav-drawer-item`
   label-with-toggle pattern: monitor icon, text "Desktop mode", and the
   existing checkbox id `toggle-compact-mode`. Keeping the id means
   `applyCompactMode()` / `initCompactModeToggle()` need no changes, and the
   localStorage persistence carries over.
2. **Visibility (CSS-only):** the row has class `nav-drawer-desktop-mode-row`,
   `display: none` by default and `display: flex` inside
   `@media (min-width: 900px)` — the same breakpoint the desktop-mode layout
   CSS already uses. Resizing below 900px hides it live; no JS.
3. **Removed from Settings:** the Desktop mode toggle-row and its hint text
   are deleted from the Display settings group (moved, not duplicated).
4. **Tour text:** the Settings tour step no longer mentions "a display mode
   optimised for desktop PCs" since the toggle no longer lives in Settings.

## Verification

Playwright:

1. 1400px viewport: open drawer → Desktop mode row visible below Dark mode;
   clicking it toggles `body.compact-mode` and persists to localStorage.
2. 400px viewport: drawer contains no Desktop mode row (display none).
3. Settings panel no longer contains the toggle.
