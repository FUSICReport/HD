# Workflow View: Summary Panel, Haemodynamics Removal

**Date:** 2026-06-10
**Status:** Approved

## Problem

Workflow view has no way to complete the report: the Summary & conclusions
section (ten FUSIC HD questions, overall impression, recommended actions)
exists only in Report view. Meanwhile the Haemodynamics panel duplicates
fields that already appear in A5C/A3C (LVOT Diameter/VTI, VTI Δ respiration/
intervention, PLR/Fluids/Pressor/Inotropes, intervention details).

## Design

1. **Remove Haemodynamics:** delete the `wv-haemo` entry from `WV_WINDOWS`.
   Panel numbering and the section navigator derive from the array, so both
   update automatically.
2. **Add final panel** `{ id: 'wv-summary', title: 'Summary & conclusions',
   summarySection: true }`. New `wvMountSummarySection(panel)` follows the
   existing moved-DOM pattern (demographics/pericardium/lungs/scan-info):
   moves `#sec-summary .section-body` into `#wv-summary-mount`, records the
   original parent in `_wvSummaryBodyParent`, and `wvTeardown()` restores it.
   The proxy pattern is deliberately not used: the FUSIC HD questions carry
   conditional show/hide logic that must travel with the real DOM.
3. **CSS:**
   - `#wv-summary:not(.wv-panel-collapsed) > .wv-panel-body { display: block;
     padding: 0 }` — same rule the other moved-DOM panels use.
   - Desktop single-column summary rules (`body.compact-mode #sec-summary
     .section-body` and children) extended to also match
     `#wv-summary-mount .section-body`.
4. **No other changes:** session-draft save/restore already collects from
   `#windowView`; exports read fields by id, which survive relocation.

## Verification

Playwright:

1. Workflow view's last panel is Summary & conclusions containing the FUSIC
   HD questions and impression/actions fields; no Haemodynamics panel.
2. Two-way data flow: answer a FUSIC question + type an impression in
   workflow view, switch to report view → values present; edit in report
   view, switch back → values present.
3. Collapse/expand and section-navigator jump for the new panel.
4. Desktop (compact) and mobile screenshots of the new panel.
