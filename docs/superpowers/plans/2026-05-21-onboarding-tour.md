# Onboarding Tour Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an on-demand spotlight tour to the FUSIC HD Report Generator that walks users through 7 key areas of the app via a dark overlay with a cutout highlight and a floating description card.

**Architecture:** All code lives in `index.html` (single-file app). CSS is added in the existing `<style>` block near the header styles. HTML tour elements are inserted after `</header>`. JS is added in the main `<script>` block. No external dependencies.

**Tech Stack:** Vanilla HTML/CSS/JS. `ResizeObserver` for repositioning on resize. `getBoundingClientRect()` for spotlight placement.

---

## Task 1: CSS — Tour elements

**Files:**
- Modify: `index.html` — `<style>` block, after `.header-actions` button styles (around line 161)

- [ ] **Step 1: Add tour CSS** after the line `.header-actions { display: flex; align-items: center; gap: 2px; }` and the existing header button rules. Find this block:

```css
#refRangesBtn, #aboutBtn, #themeToggleBtn, #settingsBtn { background: none; border: none; cursor: pointer; width: 40px; height: 40px; display: flex; align-items: center; justify-content: center; border-radius: 50%; color: var(--text-muted); transition: background 0.2s, color 0.2s; }
#refRangesBtn:hover, #aboutBtn:hover, #themeToggleBtn:hover, #settingsBtn:hover { background: var(--primary-xlight); color: var(--primary); }
#refRangesBtn svg, #aboutBtn svg, #themeToggleBtn svg, #settingsBtn svg { width: 22px; height: 22px; flex-shrink: 0; }
```

Extend the selector lists to include `#tourBtn`:

```css
#refRangesBtn, #aboutBtn, #tourBtn, #themeToggleBtn, #settingsBtn { background: none; border: none; cursor: pointer; width: 40px; height: 40px; display: flex; align-items: center; justify-content: center; border-radius: 50%; color: var(--text-muted); transition: background 0.2s, color 0.2s; }
#refRangesBtn:hover, #aboutBtn:hover, #tourBtn:hover, #themeToggleBtn:hover, #settingsBtn:hover { background: var(--primary-xlight); color: var(--primary); }
#refRangesBtn svg, #aboutBtn svg, #tourBtn svg, #themeToggleBtn svg, #settingsBtn svg { width: 22px; height: 22px; flex-shrink: 0; }
```

- [ ] **Step 2: Add the tour overlay/spotlight/tooltip CSS** immediately after those header button rules:

```css


/* ─── ONBOARDING TOUR ─────────────────────────────────── */
#tour-overlay {
  position: fixed; inset: 0; z-index: 1100;
  pointer-events: none; opacity: 0;
  transition: opacity 0.25s;
}
#tour-overlay.tour-active { pointer-events: all; opacity: 1; }
#tour-spotlight {
  position: fixed; z-index: 1101;
  box-shadow: 0 0 0 9999px rgba(0,0,0,0.55);
  border-radius: 8px;
  transition: top 0.25s ease, left 0.25s ease, width 0.25s ease, height 0.25s ease, border-radius 0.25s ease;
  pointer-events: none;
}
#tour-tooltip {
  position: fixed; z-index: 1102;
  background: var(--surface);
  border: 1.5px solid var(--border);
  border-radius: var(--radius);
  box-shadow: var(--shadow-lg);
  padding: 16px 18px;
  width: 300px;
  pointer-events: all;
  box-sizing: border-box;
}
.tour-counter {
  font-size: 11px; font-weight: 600; color: var(--text-muted);
  letter-spacing: 0.06em; text-transform: uppercase; margin-bottom: 6px;
}
.tour-title {
  font-size: 15px; font-weight: 700; color: var(--primary-dark);
  margin-bottom: 7px; line-height: 1.3;
}
.tour-desc {
  font-size: 13px; color: var(--text); line-height: 1.5; margin-bottom: 14px;
}
.tour-actions {
  display: flex; align-items: center; gap: 6px;
}
.tour-skip {
  margin-right: auto; font-size: 12px; color: var(--text-muted);
  background: none; border: none; cursor: pointer; padding: 2px 0;
  text-decoration: underline; flex-shrink: 0;
}
.tour-skip:hover { color: var(--text); }
html[data-theme="dark"] #tour-spotlight { box-shadow: 0 0 0 9999px rgba(0,0,0,0.7); }
```

- [ ] **Step 3: Verify** — Open `index.html` in a browser. No visual change yet. No console errors on load.

---

## Task 2: HTML — Tour trigger button

**Files:**
- Modify: `index.html` — `<header id="appHeader">` section (around line 3085)

- [ ] **Step 1: Add the tour button** to `.header-actions`, before `#aboutBtn`. Find:

```html
    <button type="button" id="aboutBtn" title="About" aria-label="About this application">
```

Insert before it:

```html
    <button type="button" id="tourBtn" title="Take the tour" aria-label="Start onboarding tour">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><circle cx="12" cy="12" r="10"/><polygon points="16.24 7.76 14.12 14.12 7.76 16.24 9.88 9.88 16.24 7.76"/></svg>
    </button>
```

(This is a compass/navigation icon — distinct from the existing info `?` icon.)

- [ ] **Step 2: Verify** — Reload the page. A compass icon button appears in the header between the reference-ranges button and the about button. It has hover styling matching the other header buttons.

---

## Task 3: HTML — Tour overlay elements

**Files:**
- Modify: `index.html` — after `</header>` (around line 3099), before `<div id="overlay">`

- [ ] **Step 1: Add the tour overlay HTML**. Find the line:

```html
<div id="overlay"></div>
```

Insert before it:

```html
<!-- ══════════ ONBOARDING TOUR ════════════════════════ -->
<div id="tour-overlay" aria-hidden="true">
  <div id="tour-spotlight"></div>
  <div id="tour-tooltip" role="dialog" aria-label="Tour step" aria-modal="false">
    <div class="tour-counter" id="tourCounter"></div>
    <div class="tour-title" id="tourTitle"></div>
    <div class="tour-desc" id="tourDesc"></div>
    <div class="tour-actions">
      <button class="tour-skip" id="tourSkipBtn">Skip tour</button>
      <button class="btn btn-secondary" id="tourBackBtn">Back</button>
      <button class="btn btn-primary" id="tourNextBtn">Next</button>
    </div>
  </div>
</div>

```

- [ ] **Step 2: Verify** — Reload. No visible change (overlay is invisible). Inspect the DOM to confirm `#tour-overlay`, `#tour-spotlight`, `#tour-tooltip` all exist.

---

## Task 4: JS — Tour step definitions and state

**Files:**
- Modify: `index.html` — main `<script>` block, after the `// ─── SETTINGS ──` section (around the `openSettings` / `closeSettings` functions)

- [ ] **Step 1: Add tour step definitions and state variables**. Find the line:

```javascript
function openSettings() { document.getElementById('settingsPanel').classList.add('open'); document.getElementById('overlay').classList.add('open'); document.body.style.overflow = 'hidden'; }
```

Insert the following block immediately before it:

```javascript
// ─── ONBOARDING TOUR ───────────────────────────────────
const TOUR_STEPS = [
  {
    sel: '#appHeader',
    title: 'Welcome to FUSIC HD',
    desc: 'This tool generates structured FUSIC HD echo reports. Use the icons in the top-right to switch theme, open settings, or restart this tour at any time.',
    r: '0px'
  },
  {
    sel: '#scanInfoCard',
    title: 'Scan Information',
    desc: 'Start here — enter patient demographics, operator details, machine information, and scan date/time before completing the rest of the form.',
    r: '10px'
  },
  {
    sel: '#sec-demographics',
    title: 'Form Sections',
    desc: 'The main body is divided into collapsible sections — Left Ventricle, Right Heart, Valves, and more. Click any section header to expand or collapse it.',
    r: '10px'
  },
  {
    sel: '#sum-cond-master-wrap',
    title: 'FUSIC HD Questions',
    desc: 'These ten structured questions are the heart of the FUSIC HD protocol. Answer each one to build a structured clinical summary.',
    r: '8px',
    before() {
      const sec = document.getElementById('sec-summary');
      if (sec && sec.classList.contains('section-collapsed')) toggleSection('sec-summary');
    }
  },
  {
    sel: '#summary-overall-impression-wrap',
    title: 'Clinical Conclusions',
    desc: 'Document your overall impression and recommended actions here. These fields feed directly into every export format — plain text, PDF, Excel, and more.',
    r: '8px'
  },
  {
    sel: '#settingsPanel',
    title: 'Settings & Options',
    desc: 'Customise your experience — toggle which sections appear, adjust export behaviour, enable guidance mode, and more.',
    r: '0px',
    before() { openSettings(); },
    after() { closeSettings(); }
  },
  {
    sel: '#stickyFooter',
    title: 'Exporting Your Report',
    desc: 'When ready, use the Export button to generate your report. Choose from plain text, print/PDF, Excel workbook, BSE/EDEC style, and more.',
    r: '0px'
  }
];

let _tourActive = false;
let _tourStep = 0;
let _tourRO = null;
```

- [ ] **Step 2: Verify** — Reload. No console errors. `TOUR_STEPS` is accessible in the browser console (`TOUR_STEPS.length === 7`).

---

## Task 5: JS — Core tour functions

**Files:**
- Modify: `index.html` — same `<script>` block, directly after the state variables from Task 4

- [ ] **Step 1: Add `startTour`, `endTour`, `tourNext`, `tourBack`, `_goToTourStep`, `_positionTour`**. Insert the following immediately after the `let _tourRO = null;` line:

```javascript
function startTour() {
  _tourStep = 0;
  _tourActive = true;
  const ov = document.getElementById('tour-overlay');
  ov.classList.add('tour-active');
  ov.setAttribute('aria-hidden', 'false');
  _goToTourStep(0);
}

function endTour() {
  const step = TOUR_STEPS[_tourStep];
  if (step && step.after) step.after();
  _tourActive = false;
  const ov = document.getElementById('tour-overlay');
  ov.classList.remove('tour-active');
  ov.setAttribute('aria-hidden', 'true');
  if (_tourRO) { _tourRO.disconnect(); _tourRO = null; }
}

function tourNext() {
  const step = TOUR_STEPS[_tourStep];
  if (step && step.after) step.after();
  if (_tourStep < TOUR_STEPS.length - 1) {
    _tourStep++;
    _goToTourStep(_tourStep);
  } else {
    endTour();
  }
}

function tourBack() {
  const step = TOUR_STEPS[_tourStep];
  if (step && step.after) step.after();
  if (_tourStep > 0) {
    _tourStep--;
    _goToTourStep(_tourStep);
  }
}

function _goToTourStep(idx) {
  const step = TOUR_STEPS[idx];
  if (!step) return;
  if (step.before) step.before();

  document.getElementById('tourCounter').textContent = `Step ${idx + 1} of ${TOUR_STEPS.length}`;
  document.getElementById('tourTitle').textContent = step.title;
  document.getElementById('tourDesc').textContent = step.desc;
  const backBtn = document.getElementById('tourBackBtn');
  const nextBtn = document.getElementById('tourNextBtn');
  backBtn.style.display = idx === 0 ? 'none' : '';
  nextBtn.textContent = idx === TOUR_STEPS.length - 1 ? 'Finish' : 'Next';

  const target = document.querySelector(step.sel);
  if (!target) return;

  const noScroll = step.sel === '#appHeader' || step.sel === '#stickyFooter';
  if (!noScroll) target.scrollIntoView({ behavior: 'smooth', block: 'center' });

  // Delay positioning to let scroll and any panel animations settle
  const delay = step.sel === '#settingsPanel' ? 400 : 350;
  setTimeout(() => { if (_tourActive) _positionTour(target, step); }, delay);

  if (_tourRO) _tourRO.disconnect();
  _tourRO = new ResizeObserver(() => { if (_tourActive) _positionTour(target, step); });
  _tourRO.observe(document.body);
}

function _positionTour(target, step) {
  const rect = target.getBoundingClientRect();
  const pad = 6;
  const sl = document.getElementById('tour-spotlight');
  sl.style.top    = `${rect.top    - pad}px`;
  sl.style.left   = `${rect.left   - pad}px`;
  sl.style.width  = `${rect.width  + pad * 2}px`;
  sl.style.height = `${rect.height + pad * 2}px`;
  sl.style.borderRadius = step.r || '8px';

  const tt = document.getElementById('tour-tooltip');
  const ttW = 300;
  const margin = 14;
  const vpH = window.innerHeight;
  const vpW = window.innerWidth;

  const spaceBelow = vpH - rect.bottom - pad;
  const spaceAbove = rect.top - pad;
  let top;
  if (spaceBelow >= 180 || spaceBelow >= spaceAbove) {
    top = rect.bottom + pad + margin;
  } else {
    top = rect.top - pad - margin - tt.offsetHeight;
  }
  top = Math.max(8, Math.min(top, vpH - tt.offsetHeight - 8));

  let left = rect.left + rect.width / 2 - ttW / 2;
  left = Math.max(8, Math.min(left, vpW - ttW - 8));

  tt.style.top  = `${top}px`;
  tt.style.left = `${left}px`;
}
```

- [ ] **Step 2: Verify** — Reload. Functions `startTour`, `endTour`, `tourNext`, `tourBack` are accessible in the browser console. Calling `startTour()` in the console should show the tour overlay (though buttons aren't wired yet).

---

## Task 6: JS — Wire up event listeners

**Files:**
- Modify: `index.html` — `<script>` block, inside the existing `DOMContentLoaded` / init section. Find where other button listeners are attached at init time (search for `document.getElementById('scanInfoCollapseAllBtn')?.addEventListener`).

- [ ] **Step 1: Add tour button and overlay event listeners**. Find this line in the init block:

```javascript
  document.getElementById('scanInfoCollapseAllBtn')?.addEventListener('click', toggleAllCollapsibleSections);
```

Add these lines immediately after it:

```javascript
  document.getElementById('tourBtn')?.addEventListener('click', startTour);
  document.getElementById('tourSkipBtn')?.addEventListener('click', endTour);
  document.getElementById('tourBackBtn')?.addEventListener('click', tourBack);
  document.getElementById('tourNextBtn')?.addEventListener('click', tourNext);
  document.getElementById('tour-overlay')?.addEventListener('click', function(e) {
    if (e.target === this) endTour();
  });
```

- [ ] **Step 2: Verify** — Reload. Click the compass icon in the header. The tour overlay appears with the spotlight on the header and a tooltip below it reading "Step 1 of 7 / Welcome to FUSIC HD". The Back button is hidden. Clicking "Next" advances to step 2. Clicking "Skip tour" or the dark overlay area closes the tour.

- [ ] **Step 3: Verify all 7 steps** — Walk through the entire tour:
  - Step 1: Header highlighted, tooltip below it.
  - Step 2: Scan Information card highlighted, tooltip below.
  - Step 3: Patient Demographics section highlighted.
  - Step 4: Summary section auto-expands if collapsed; FUSIC HD Questions block highlighted.
  - Step 5: Overall Impression field highlighted.
  - Step 6: Settings panel slides open; panel highlighted; panel closes when leaving step.
  - Step 7: Sticky footer highlighted, Next button reads "Finish". Clicking Finish closes tour.

- [ ] **Step 4: Verify Back navigation** — Start tour, advance to step 3, click Back. Returns to step 2 correctly. On step 1, Back button is hidden.

- [ ] **Step 5: Verify overlay dismiss** — Start tour, click the dark area outside the spotlight. Tour closes.

- [ ] **Step 6: Verify settings panel cleanup** — Start tour, advance to step 6 (settings panel opens). Click "Skip tour" or the overlay. Settings panel closes.

- [ ] **Step 7: Verify dark mode** — Toggle to dark mode, start tour. Overlay is darker (`rgba(0,0,0,0.7)`), tooltip uses dark surface colours. All text readable.

---

## Task 7: Commit

- [ ] **Step 1: Commit all changes**

```bash
git add index.html
git commit -m "Add on-demand spotlight onboarding tour"
```
