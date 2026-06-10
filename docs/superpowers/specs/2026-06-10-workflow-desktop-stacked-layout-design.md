# Workflow View: Stacked Desktop Layout

**Date:** 2026-06-10
**Status:** Approved

## Problem

On desktop ("compact" mode), Workflow view renders as 14 narrow 200px columns
scrolling horizontally (~2950px of content), while Report view renders as a
centred 1232px page of stacked sections with 5-column field grids. The user
wants desktop Workflow view to look like desktop Report view.

## Decisions

- **Stacked wide sections** (not a restyle of the horizontal columns): each
  workflow panel (Scan info, Demographics, PLAX, PSAX, A4C, ... Haemodynamics)
  becomes a full-width section stacked vertically, ordered by echo window.
- **The horizontal column layout is removed entirely.** It existed only in
  desktop Workflow view; its code is deleted, not kept behind a toggle.

## Approach

Promote the existing stacked layout (today scoped to mobile via
`body:not(.compact-mode)`) to be the only Workflow layout, then add desktop
density overrides mirroring Report view's compact rules. CSS-only; the JS
panel builder, field proxies and collapse logic already work in stacked mode.

### 1. Base layout (all modes)

- `#windowView`: `overflow-x: hidden; overflow-y: auto;
  scrollbar-gutter: stable both-edges` (keeps scrollbar identical to Report
  view, per the 2026-06-10 scroll-container unification).
- Delete horizontal base rules: `flex-direction: row`, `gap: 10px`,
  `align-items: flex-start`, `.wv-panel { flex: 0 0 200px; max-height: 100%;
  overflow-y: auto }`, sticky panel titles.
- Un-scope all `body:not(.compact-mode)` workflow rules (stacked flex column,
  centred 688px panels, Report-style clickable headers with chevrons,
  2-column field grid, subpanel styling, dark-mode variants). Mobile must
  render identically to today.

### 2. Desktop density overrides (`@media (min-width: 900px)`, `body.compact-mode`)

Mirror Report view's compact values:

- Container padding `10px 24px 40px`; panels and page header
  `max-width: 1232px`; panel `margin-bottom: 4px`.
- Field grids: `repeat(5, minmax(0, 1fr))`, gap `3px 12px`, padding
  `4px 10px 6px`. Existing full-width exceptions (textareas, checkbox rows,
  rhythm chips, subheadings, lung grid) carry over unchanged.
- Slim headers: panel title padding `4px 10px`; panel icon 18px; chevron 13px.
- Inputs tightened (the existing global `body.compact-mode input/select/
  textarea { padding: 4px 7px }` rules already win by specificity); labels 10px.

### 3. Behaviour changes

- Desktop workflow panels become collapsible (header click + collapse-all in
  the page header), like mobile workflow and Report sections.
- The workflow page header, panel icons and chevrons — previously hidden on
  desktop — are shown; the three `body.compact-mode ... { display: none }`
  rules are deleted.

### 4. Out of scope

- No changes to Report view, the JS panel builder, field proxies, or mobile
  Workflow rendering.

## Verification

Playwright (headless Chromium, file://):

1. Desktop compact Workflow: panels stacked vertically at 1232px, no
   horizontal scroll, 5-column grids — visual match with Report view density.
2. Mobile/non-compact Workflow: pixel-identical before/after screenshots.
3. Collapse/expand a panel and collapse-all on desktop.
4. Section navigator jump in desktop Workflow.
5. Report ↔ Workflow toggle round-trip in both modes.
