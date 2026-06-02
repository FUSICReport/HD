---
name: burger-menu-header-redesign
description: Centre the app title in the header and replace the right-side action buttons with a left-side burger menu that opens a slide-in drawer
metadata:
  type: project
---

## Overview

Redesign the `#appHeader` so the title "FUSIC HD Report Generator" is visually centred, and the three existing action buttons (Reference Ranges, About, Settings) are moved into a slide-in drawer opened by a burger icon on the left.

## Header Layout

The header becomes a 3-column flex layout:

```
[ burger button (40px) ] [ centred title (flex-grow: 1) ] [ spacer (40px) ]
```

- The burger button and the right spacer are the same fixed width so the title is mathematically centred.
- The `.header-logo` div and `.header-actions` div are replaced: `.header-logo` becomes a single centred `<span class="header-title">`, and `.header-actions` is removed.
- The existing `header-title` CSS is updated to `text-align: center`.

## Burger Button

- ID: `burgerBtn`
- Positioned as the leftmost element in the header.
- Standard 3-line SVG icon, same 40px / 48px (touch) size and hover style as the existing action buttons.
- `aria-label="Open menu"`, `aria-expanded` toggled on open/close.

## Slide-in Drawer

### Structure
- `<div id="navDrawer">` — the drawer panel, slides in from the left.
- `<div id="navDrawerBackdrop">` — full-screen semi-transparent overlay behind the drawer.

### Appearance
- Width: 260px (desktop), 80vw (mobile ≤520px).
- Background: `var(--surface)`, right-side border: `2px solid var(--primary-light)` (matches header bottom border).
- Box shadow to the right.
- Z-index: 200 (above the header's z-index 100).

### Contents
Three full-width buttons stacked vertically, each with an icon + label:
1. **Reference Ranges** — same SVG icon as current `#refRangesBtn`
2. **About** — same SVG icon as current `#aboutBtn`
3. **Settings** — same SVG icon as current `#settingsBtn`

Each button retains its existing ID (`refRangesBtn`, `aboutBtn`, `settingsBtn`) so all existing JS event listeners continue to work without modification.

### Drawer header
A small `×` close button (`id="navDrawerClose"`) at the top-right inside the drawer, labelled `aria-label="Close menu"`.

### Open / Close behaviour
- Clicking `#burgerBtn` opens the drawer (adds `.open` class, shows backdrop).
- Clicking the backdrop closes it.
- Pressing `Escape` closes it.
- Focus is trapped inside the drawer while open (for accessibility).
- `aria-expanded` on `#burgerBtn` reflects open/closed state.

### Animation
CSS `transform: translateX(-100%)` → `translateX(0)` with `transition: transform 0.25s ease`.

## CSS Changes

| Selector | Change |
|---|---|
| `#appHeader` | Keep flex + space-between; children become burger, title span, spacer |
| `.header-logo` | Remove |
| `.header-actions` | Remove |
| `#refRangesBtn, #aboutBtn, #settingsBtn` | Move into drawer; update sizing to full-width |
| `.header-title` | Add `text-align: center; flex: 1` |
| New: `#burgerBtn` | 40px/48px circle button, same hover style |
| New: `.header-spacer` | 40px/48px fixed width invisible element |
| New: `#navDrawer` | Slide-in panel styles |
| New: `#navDrawerBackdrop` | Full-screen overlay |
| New: `.nav-drawer-item` | Full-width icon+label button style |

## JS Changes

- Add `burgerBtn` click handler to toggle drawer open/close.
- Add backdrop click handler to close.
- Add `keydown` Escape listener to close.
- No changes to existing button handlers (IDs are preserved).

## Files Affected

- `index.html` — HTML structure (header section ~line 3544) and inline CSS (~line 122).

## Out of Scope

- Adding new menu items.
- Changing the drawer's position (always left).
- Changing the existing modal/panel behaviours triggered by the three buttons.
