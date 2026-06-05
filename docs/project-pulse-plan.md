---
# Project Pulse Dashboard — Implementation Plan

## 1. Summary

Mona's team needs a lightweight, browser-based **Project Pulse** dashboard that lets contributors see, at a glance, which projects are active, who owns them, current status, recent activity, and priority/risk. The deliverable is a small static app rendered from JSON data, plus a VS Code launch configuration so the dashboard opens directly to the UI (not a directory listing).

**Deliverables**
- `app/index.html` — semantic structure + data-loading/rendering script
- `app/styles.css` — dashboard layout, cards, badges, responsive design
- `app/project-data.json` — sample data with a top-level `projects` array
- `.vscode/launch.json` — **Run Project Pulse Dashboard** config; `cwd` = `${workspaceFolder}/app`; opens `index.html`

**Agents**
- **Designer** owns `app/styles.css`
- **Coder** owns `app/index.html`, `app/project-data.json`, `.vscode/launch.json`

**Shared contract (must be agreed before parallel work begins)**
The two agents must share a small CSS-hook + data-shape contract so styling and rendering line up:

- Container: `<main class="dashboard">`
- Header: `<header class="dashboard__header">` with `h1` "Project Pulse"
- Grid wrapper: `<section class="dashboard__grid" id="project-grid" aria-live="polite">`
- Card: `<article class="project-card" data-priority="high|medium|low" data-status="...">`
  - `.project-card__header` (name + priority pill)
  - `.project-card__name`
  - `.project-card__owner`
  - `.project-card__status` → `<span class="status-badge status-badge--{status-slug}">`
  - `.project-card__priority` → `<span class="priority-pill priority-pill--{priority}">`
  - `.project-card__activity`
  - `.project-card__summary` (optional, contributor-friendly blurb)
- Empty state: `<p class="dashboard__empty">`
- Error state: `<p class="dashboard__error" role="alert">`

Status vocabulary (slugged for CSS class suffix): `on-track`, `at-risk`, `blocked`, `completed`, `planning`.
Priority vocabulary: `high`, `medium`, `low`.

## 2. Ordered implementation steps

### Step 1 — Lock the shared contract (Planner output, no code)
- Output: the CSS hook list, data fields, and status/priority vocabularies above.
- Owner: Orchestrator publishes the contract to both agents before work starts.
- Files: none.

### Step 2 — Author sample data
- File: `app/project-data.json`
- Owner: **Coder**
- Content: top-level `{ "projects": [ ... ] }` with 5–7 sample projects. Each project includes the required fields: `name`, `owner`, `status`, `recentActivity`, `priority`. Add a `summary` field to satisfy "short contributor-friendly summary" from the brief. Include at least one entry per status and per priority to exercise styling.
- Strict JSON, no comments, no trailing commas.

### Step 3 — Build HTML structure + data rendering
- File: `app/index.html`
- Owner: **Coder**
- Includes:
  - `<!doctype html>`, `<html lang="en">`, meta charset, viewport meta for responsiveness
  - `<title>Project Pulse</title>`
  - `<link rel="stylesheet" href="styles.css">`
  - Semantic structure matching the shared contract (`main.dashboard`, header, grid)
  - Inline `<script type="module">` (or plain `<script>`) that:
    1. `fetch('./project-data.json')`
    2. Parses JSON, validates `projects` is an array
    3. Renders cards into `#project-grid` using `document.createElement` (no `innerHTML` injection of data values — use `textContent` to avoid XSS)
    4. Slugs the status (`status.toLowerCase().replace(/\s+/g,'-')`) to produce class suffix and `data-status`
    5. Shows empty state if zero projects; shows error state on fetch/parse failure
  - No external runtime libraries.

### Step 4 — Style the dashboard
- File: `app/styles.css`
- Owner: **Designer**
- Includes:
  - CSS custom properties for color palette, spacing scale, radii, shadows
  - Base reset/typography, system font stack
  - `.dashboard` page layout with comfortable max-width and padding
  - `.dashboard__header` with title + optional subtitle/legend
  - `.dashboard__grid` using CSS Grid (`repeat(auto-fill, minmax(280px, 1fr))`) with gap
  - `.project-card` with rounded corners, shadow, padding, hover/focus affordance
  - `.status-badge--*` color-coded per status (on-track=green, at-risk=amber, blocked=red, completed=blue/neutral, planning=purple/neutral)
  - `.priority-pill--high|medium|low` with distinct, accessible color contrast
  - Visual priority cue on the card itself (e.g., left border accent via `[data-priority="high"]`)
  - Responsive breakpoints: ≤480px single column, condensed padding; ≥768px multi-column; ≥1200px wider gutter
  - Accessibility: WCAG AA contrast, visible focus ring, `prefers-reduced-motion` respected, `prefers-color-scheme: dark` optional but recommended
  - Empty/error state styling

### Step 5 — VS Code launch configuration
- File: `.vscode/launch.json`
- Owner: **Coder**
- Requirements:
  - `name`: "Run Project Pulse Dashboard"
  - Opens `index.html` in a browser, served (or opened) from `app/`
  - `cwd`: `${workspaceFolder}/app`
  - Strict JSON, no comments
  - Recommended approach: use a `node` type that runs `python3 -m http.server 4173` with `cwd: ${workspaceFolder}/app`, plus `serverReadyAction` to open `http://localhost:4173/index.html`.

### Step 6 — Integration validation
- Owner: Orchestrator coordinates; both agents available for fixes
- Run the launch config, confirm the dashboard renders all sample projects with correct badges, priorities, and responsive behavior.

## 3. Dependencies between steps

- Step 1 (contract) → blocks Steps 2, 3, 4, 5
- Step 2 (data) → blocks Step 3's runtime validation (Coder needs real data to test render); Step 3 code can be written in parallel with Step 2 because the shape is fixed by Step 1
- Step 3 (HTML/JS) and Step 4 (CSS) are independent in file scope and can run in parallel once Step 1 is locked
- Step 5 (launch.json) is independent of Steps 3 and 4 but depends on the decision about whether a static server is needed (decided in Step 1 / Step 3 design)
- Step 6 (integration validation) depends on Steps 2, 3, 4, 5 all complete

## 4. Parallel work decisions

The following can run **concurrently** because they touch disjoint files and rely only on the Step 1 contract:

- **Coder track:** Step 2 (`project-data.json`) → Step 3 (`index.html`) → Step 5 (`.vscode/launch.json`)
- **Designer track:** Step 4 (`styles.css`)

The Designer can style against the agreed class hooks without waiting for the Coder, using the documented markup contract. The Coder can render markup without waiting for the Designer, because class names are pre-agreed.

## 5. Sequential work decisions

- Step 1 must complete before any other step (defines the shared contract).
- Step 6 (integration validation) must be last — it cannot start until all artifacts exist.
- Within the Coder track, Step 3 should ideally land after Step 2 so the Coder can smoke-test rendering with real data; if interleaved, Step 3 may proceed with a stub that matches the contract.
- Any change to the shared contract after Step 1 must be re-broadcast to both agents before they continue, to prevent class-name drift.

## 6. Edge cases to handle

**Data**
- Empty `projects` array → render `.dashboard__empty` message ("No projects to show yet.")
- Missing optional field (e.g., `summary`) → omit gracefully, no "undefined" text
- Missing required field → render card with placeholder ("—") and `console.warn`
- Unknown `status` or `priority` value → fall back to neutral badge styling; do not crash
- Very long `name` or `recentActivity` → CSS must wrap (no horizontal overflow); consider `overflow-wrap: anywhere` and a max line clamp for activity
- Special characters / HTML in data → must be inserted via `textContent`, never `innerHTML`
- Large list (e.g., 50 projects) → grid must scroll naturally; no fixed-height pitfalls

**Network / runtime**
- `fetch` fails (file:// scheme or server down) → show `.dashboard__error` with actionable hint ("Run the *Run Project Pulse Dashboard* launch config.")
- Malformed JSON → caught, error state shown
- Browser without `fetch` (none modern lack it, but document minimum: evergreen browsers)

**Responsive / accessibility**
- Mobile ≤360px width — cards remain readable, badges don't overlap
- Keyboard navigation — cards focusable only if interactive; otherwise headings provide structure
- Color is not the sole signal for status/priority — include text label inside badge/pill
- Respect `prefers-reduced-motion` (no hover scale animations)
- Respect `prefers-color-scheme: dark` if dark theme implemented

**Tooling**
- `.vscode/launch.json` must be strict JSON (no comments)
- Port collision for static server — choose an uncommon port (e.g., 4173) or document override

## 7. Validation expectations

**`app/project-data.json`**
- Parses with `JSON.parse` (no comments, no trailing commas)
- Top-level key is `projects`, an array of ≥5 objects
- Every object has `name`, `owner`, `status`, `recentActivity`, `priority`
- Covers each status value and each priority value at least once

**`app/index.html`**
- Validates as HTML5 (no unclosed tags, has `<!doctype html>` and `lang`)
- Loads `styles.css` and renders without console errors
- Cards in DOM match the count in `project-data.json`
- Each card contains the required fields visibly
- Empty-state and error-state code paths exercised by temporarily emptying / breaking the JSON

**`app/styles.css`**
- `.dashboard` and `.project-card` selectors exist
- Distinct visual treatment per status and per priority (manual visual check)
- Responsive: looks correct at 360px, 768px, 1280px viewport widths
- Lighthouse Accessibility ≥ 95; axe DevTools shows no contrast violations on badges/pills
- Visible focus indicator on any focusable element

**`.vscode/launch.json`**
- Parses as strict JSON
- Configuration named exactly **Run Project Pulse Dashboard**
- `cwd` resolves to `${workspaceFolder}/app`
- Launching it opens the dashboard UI (not a directory listing)
- `fetch('./project-data.json')` succeeds at runtime

**Integration**
- From a fresh clone, opening the workspace and pressing F5 → choosing the config → dashboard renders all sample projects with correct badges and responsive layout.

## 8. Open questions

1. **Static server vs. file://** — Because the brief requires a separate `project-data.json` loaded by the page, `fetch()` will fail under `file://`. Plan assumes the Coder will run a local static server in the launch config (`python3 -m http.server` is available in the devcontainer). Confirm Python is available, or fall back to Node's `npx http-server`.
2. **Dark mode** — Brief doesn't require it. Designer should treat as optional polish, not a blocker.
3. **Sorting / filtering** — Brief doesn't request interactive controls. Plan excludes them; revisit if Mona asks.
4. **Branding** — No logo or color guide supplied. Designer chooses a neutral, professional palette; document choices in completion report.
5. **Status vocabulary** — Plan proposes 5 values. Confirm with Mona, or let Designer/Coder finalize from the sample data they author.
6. **Number of sample projects** — Plan suggests 5–7 to exercise all states. Confirm acceptable.
7. **Accessibility target** — Plan targets WCAG AA. Confirm no AAA requirement.

---

**Phasing summary for Orchestrator:**
- **Phase A (sequential):** Step 1 — broadcast contract
- **Phase B (parallel):** Coder runs Steps 2 → 3 → 5 concurrently with Designer running Step 4
- **Phase C (sequential):** Step 6 — integration validation
