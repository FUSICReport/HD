# Onboarding Tour — Design Spec
_Date: 2026-05-21_

## Overview

An on-demand interactive spotlight tour that walks new (and returning) users through the key areas of the FUSIC HD Report Generator. The tour is triggered manually and uses a spotlight overlay to highlight one UI element at a time alongside a description card.

---

## Trigger

- A **tour icon button** (e.g. graduation-cap or compass icon) is added to the app header alongside the existing theme/settings icon buttons.
- Clicking it starts the tour from step 1 at any time.
- No auto-show on first visit; no localStorage flag needed.

---

## Interaction Model

### Overlay & Spotlight
- A **full-screen fixed overlay** (`position: fixed; inset: 0`) with a semi-transparent dark background (`rgba(0,0,0,0.55)`).
- A **highlight element** (`div#tour-spotlight`) is positioned absolutely over the target element's bounding rect (obtained via `getBoundingClientRect()`). It uses a large `box-shadow` spread (`box-shadow: 0 0 0 9999px rgba(0,0,0,0.55)`) to create the cutout effect. The element itself is transparent.
- The highlight div has a `border-radius` that matches the target element's visual style (e.g. 8px for cards, 50% for icon buttons).
- On step change, the highlight div smoothly repositions via CSS `transition` on `top`, `left`, `width`, `height`.

### Description Card (Tooltip)
- A **floating card** (`div#tour-tooltip`) positioned near the highlighted element — below by default, above if the element is in the lower half of the screen.
- Contains: step title, description text, step counter ("3 of 7"), and Back / Next (or Finish) / Skip controls.
- Card stays within viewport bounds; its position is clamped so it never overflows left/right edges.

### Controls
- **Back** — go to previous step (hidden on step 1)
- **Next / Finish** — advance to next step; on last step the button reads "Finish" and closes the tour
- **Skip** — closes the tour immediately from any step
- Clicking anywhere on the dark overlay outside the spotlight also dismisses the tour

---

## Tour Steps (7)

| # | Target element | Title | Description |
|---|---|---|---|
| 1 | `#appHeader` | Welcome to FUSIC HD | Brief intro to the app — a structured echo reporting tool. Describes the header controls: theme toggle, settings, and how to start the tour again. |
| 2 | `#scanInfoCard` | Scan Information | Start here. Enter patient demographics, operator details, machine info, and scan date/time before completing the rest of the form. |
| 3 | `#sec-demographics` | Form Sections | The main body is split into collapsible sections (e.g. Left Ventricle, Right Heart, Valves). Click a section header to expand or collapse it. |
| 4 | `#sum-cond-master-wrap` | FUSIC HD Questions | The ten structured questions at the heart of the FUSIC HD protocol, answered in the Summary section. `#sec-summary` is expanded programmatically before this step if collapsed. |
| 5 | `#summary-overall-impression-wrap` | Clinical Conclusions | Document your overall impression and recommended actions here. These feed directly into every export format. |
| 6 | `#settingsPanel` | Settings & Options | The settings panel (opened automatically for this step) lets you toggle sections, adjust export behaviour, enable guidance mode, and more. Closed automatically when moving on. |
| 7 | `#stickyFooter` | Exporting Your Report | When ready, use the Export button to choose from multiple formats: plain text, print/PDF, Excel workbook, BSE/EDEC style report, and more. |

---

## Summary Section Step (step 4)

- Steps 4 and 5 both target elements inside `#sec-summary`. Before entering step 4, the tour checks if `#sec-summary` is collapsed and calls `toggleSection('sec-summary')` to expand it if so.
- The section is left expanded when moving on (the user may want to fill it in immediately).

---

## Settings Panel Step (step 6)

- When the tour reaches step 6, `openSettings()` is called programmatically to slide the panel into view.
- The spotlight targets `#settingsPanel`.
- When the user navigates away (Back or Next) or skips, `closeSettings()` is called.

---

## Scrolling

- Before positioning the spotlight on a target, the tour calls `element.scrollIntoView({ behavior: 'smooth', block: 'center' })` so the element is visible.
- For the header (step 1) and footer/`#stickyFooter` (step 7), no scroll is needed — they are always visible.

---

## Implementation Details

### New elements
- `div#tour-overlay` — full-screen fixed dark overlay, `z-index: 1100` (above all existing UI)
- `div#tour-spotlight` — positioned highlight box inside the overlay
- `div#tour-tooltip` — description card, also inside the overlay

### State
- `let tourActive = false`
- `let tourStep = 0` (0-indexed)
- Tour step definitions: a plain JS array of objects `{ targetSelector, title, description, beforeEnter?, afterLeave? }`. `beforeEnter` / `afterLeave` are optional callbacks (used for the settings panel step).

### No external dependencies
Everything is vanilla JS/CSS, consistent with the rest of the app.

### Dark mode
The tour card uses existing CSS variables (`--surface`, `--text`, `--primary`, `--border`) so it adapts automatically.

### Resize / scroll
A `ResizeObserver` (or fallback `window.resize` listener) repositions the spotlight while the tour is active.

---

## CSS

- Tour elements are hidden (`display: none` or `opacity: 0; pointer-events: none`) when the tour is inactive.
- The spotlight uses `transition: all 0.25s ease` for smooth repositioning between steps.
- The tooltip card style reuses existing card/modal visual language (border-radius, box-shadow, surface colour).

---

## Out of Scope

- Auto-show on first visit (explicitly excluded — on-demand only)
- Per-field micro-tutorials (that is the existing Guidance Mode feature)
- Persistence of "completed" state
- Mobile-specific layout changes (the tour works on mobile but tooltip placement uses the same logic)
