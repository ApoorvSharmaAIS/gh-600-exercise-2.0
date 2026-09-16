# Project Pulse — Implementation Plan

## 1. Overview & goals

**Project Pulse** is a lightweight, static, front-end-only dashboard that helps Mona's contributors understand project health at a glance. It renders project cards from a local JSON file using plain HTML + CSS (no framework, no build step) and is runnable from VS Code via a **Run Project Pulse Dashboard** launch configuration.

### Goals

- Show contributors, at a glance: which projects are active, who owns them, current status, recent activity, and priority/risk.
- Keep the stack intentionally simple: `index.html` + `styles.css` + `project-data.json`, served by `python3 -m http.server`.
- Provide a polished card-based UI with status badges, clear hierarchy, and accessible markup.
- Run locally with one click from VS Code's Run and Debug panel, opening the dashboard UI (not a directory listing).

### Non-goals

- No backend, no build tooling, no npm dependencies.
- No authentication, no live data sources — sample JSON only.
- No SPA framework; vanilla `fetch` + DOM is sufficient.

### Repo state assumption

Greenfield for the app surface: `app/` does not exist yet and `.vscode/launch.json` does not exist yet. `.vscode/tasks.json` already exists and must not be disturbed.

---

## 2. Designer responsibilities

**Owns:** `app/styles.css` (exclusive).
**Consults on:** semantic structure inside `app/index.html` (class names, landmarks) — but does not edit it.

### Visual language

- Define design tokens as CSS custom properties on `:root`: color palette (surface, text, muted, accent, plus status colors for `on-track` / `at-risk` / `blocked` / `done`), spacing scale, radii, shadow scale, font stack.
- Typography: system font stack, clear type ramp for dashboard title, card title, meta text.
- Priority/status treatment: pill-shaped status badges with color + text (never color alone — accessibility).

### Layout

- Top-level `.dashboard` container providing max-width, centered layout, and page padding. **(Required selector — validation gate.)**
- `.project-card` component with `border-radius`, `box-shadow`, internal padding, and a consistent grid of fields. **(Required selector + properties — validation gate.)**
- Responsive grid of cards using CSS Grid `repeat(auto-fill, minmax(...))` — 1 column on narrow viewports, multi-column on wider viewports.
- Header region for the "Project Pulse" title and a short summary line.

### Accessibility

- Color contrast ≥ WCAG AA for text and badges.
- Status conveyed by text label + icon/shape, not color alone.
- Visible focus rings on any interactive element.
- Respect `prefers-reduced-motion` for any transitions.
- Ensure the card grid remains readable at 320px width and up to 1440px+.

### States to style

- Default card, hover (subtle elevation change), keyboard focus.
- Each status variant (`.status--on-track`, `.status--at-risk`, `.status--blocked`, `.status--done`).
- Each priority variant (`.priority--high`, `.priority--medium`, `.priority--low`).
- Empty state (in case `projects` array is empty).

### Files owned by Designer

- `app/styles.css`

---

## 3. Coder responsibilities

**Owns:** `app/index.html`, `app/project-data.json`, `.vscode/launch.json` (exclusive).
**Consults on:** class-name contract with Designer so CSS selectors match markup.

### `app/index.html` — markup & data wiring

- Exact page title text **"Project Pulse"** (validation gate).
- `<link rel="stylesheet" href="styles.css">` (validation gate: must reference `styles.css`).
- Root wrapper element with class `dashboard`.
- Semantic landmarks: `<header>`, `<main>`, `<footer>` (optional).
- A script (inline or `<script src>`) that `fetch`es `project-data.json` and renders one `<article class="project-card">` per entry (validation gate: must include the phrase `project-card` and reference `project-data.json`).
- Each rendered card must display the project's `name`, `owner`, `status`, `recentActivity`, and `priority` values (validation gate).
- Status and priority rendered via badge elements with modifier classes matching Designer's contract.
- Graceful empty-state message if `projects` is empty; graceful error message if fetch fails.
- Include the phrase `index.html` somewhere referenced (already covered by the launch URL comment/config, and by title/anchor if applicable — validation gate.)

### `app/project-data.json` — schema & sample records

- Strict JSON (no comments, no trailing commas) so `python3 -m json.tool` parses cleanly.
- Top-level key `"projects"` whose value is an array. **(Validation gate.)**
- Each project object includes exactly these fields (at minimum):
  - `name` (string)
  - `owner` (string)
  - `status` (string; recommend enum: `"on-track" | "at-risk" | "blocked" | "done"`)
  - `recentActivity` (string; short human-readable sentence)
  - `priority` (string; recommend enum: `"high" | "medium" | "low"`)
- Provide 4–6 sample records covering multiple status and priority values so the UI demonstrates each badge variant.

Example shape:

```json
{
  "projects": [
    {
      "name": "Octo Onboarding",
      "owner": "@mona",
      "status": "on-track",
      "recentActivity": "Merged PR #42 improving the welcome flow",
      "priority": "high"
    }
  ]
}
```

### `.vscode/launch.json` — VS Code launch configuration

- Strict JSON, **no comments** (validation gate: `python3 -m json.tool .vscode/launch.json` must succeed).
- One configuration named exactly **"Run Project Pulse Dashboard"** (validation gate).
- Uses Node's built-in ability or `type: "node"` with `runtimeExecutable: "python3"` to run `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app` so the server serves from the `app/` directory (validation gate: must serve from app directory).
- `serverReadyAction` configured with a `pattern` that matches the http.server "Serving HTTP on ... port ([0-9]+)" line and a `uriFormat` of `http://localhost:%s/index.html` so the browser opens the dashboard UI rather than a directory listing (validation gate).
- `console: "integratedTerminal"` so the server output is visible.

### Files owned by Coder

- `app/index.html`
- `app/project-data.json`
- `.vscode/launch.json`

---

## 4. File assignments

| File                    | Owner    | Purpose                                                                                  |
| ----------------------- | -------- | ---------------------------------------------------------------------------------------- |
| `app/index.html`        | Coder    | Dashboard markup, semantic structure, fetch + render loop for project cards.             |
| `app/styles.css`        | Designer | Visual system (tokens, typography, color), `.dashboard` and `.project-card` layout, responsive grid, badge states, accessibility styling. |
| `app/project-data.json` | Coder    | Sample project data. Top-level `projects` array; each record has `name`, `owner`, `status`, `recentActivity`, `priority`. |
| `.vscode/launch.json`   | Coder    | **Run Project Pulse Dashboard** configuration; serves from `app/` on port 5500 and opens `http://localhost:%s/index.html`. |

---

## 5. Dependencies

Ordered "must happen before" relationships:

1. **Data schema (`app/project-data.json`) before markup wiring in `app/index.html`.** The Coder needs the exact field names locked in before writing the render loop.
2. **Class-name contract (Designer ↔ Coder) before either finishes their file.** Both must agree on `.dashboard`, `.project-card`, `.status--*`, `.priority--*` names so selectors match markup. This is a lightweight upfront agreement, not a blocking artifact.
3. **Design tokens (`:root` custom properties in `styles.css`) before final component CSS.** Component rules should consume tokens rather than hardcode colors/spacing.
4. **`app/index.html` exists before `.vscode/launch.json` is meaningful to test.** The launch target opens `index.html`; without it, the launch would show a 404 or directory listing.
5. **All three app files present before end-to-end validation** (opening the dashboard via the launch configuration).

---

## 6. Parallel work decisions

Because file ownership is fully non-overlapping (Designer touches only `app/styles.css`; Coder touches `app/index.html`, `app/project-data.json`, `.vscode/launch.json`), most work can proceed in **parallel** once the class-name contract is agreed.

### Can run in parallel

- **Designer** builds `app/styles.css` (tokens → `.dashboard` layout → `.project-card` component → status/priority variants → responsive rules → accessibility polish).
- **Coder** builds `app/project-data.json` (schema + sample records) and `app/index.html` (markup + fetch/render loop).
- **Coder** drafts `.vscode/launch.json` in parallel with the HTML, since its content is independent of markup details.

Rationale: no two agents touch the same file, so there is no merge conflict risk. The only shared surface is the class-name contract, which is a one-time agreement, not an ongoing coupling.

### Must run sequentially

- **Class-name contract agreement → both Designer's CSS selectors and Coder's HTML class attributes.** Both agents block on this handshake.
- **Data schema finalized → HTML render loop wiring.** The render code references field names.
- **`app/index.html` created → meaningful test of `.vscode/launch.json`.** The launch target must resolve to a real file.
- **All files present → end-to-end validation.**

---

## 7. Validation expectations

### Automated / mechanical checks

- `python3 -m json.tool app/project-data.json` parses cleanly.
- `python3 -m json.tool .vscode/launch.json` parses cleanly (no comments, no trailing commas).
- `app/project-data.json` contains a top-level `"projects"` key whose array items each have `name`, `owner`, `status`, `recentActivity`, `priority`.
- `app/index.html` contains the exact text `Project Pulse`, references `styles.css` and `project-data.json`, and includes the class `project-card`.
- `app/styles.css` contains selectors `.dashboard` and `.project-card`, plus the properties `border-radius` and `box-shadow`.
- `.vscode/launch.json` contains a configuration named `Run Project Pulse Dashboard`, serves from the `app` directory, and opens `http://localhost:%s/index.html`.

### Manual runtime checks (via launch config)

1. Open **Run and Debug** in VS Code → select **Run Project Pulse Dashboard** → press ▶.
2. Confirm the integrated terminal shows `python3 -m http.server` starting on port 5500.
3. Confirm the browser opens directly to `http://localhost:5500/index.html` — the **dashboard UI**, not a directory listing.
4. Confirm project cards render for every entry in `project-data.json`, each showing `name`, `owner`, `status`, `recentActivity`, `priority`.
5. Open DevTools Console → verify **no console errors** (no failed `fetch`, no 404s, no CSS parse warnings).

### Visual / design checks

- Layout matches Designer's intent: card grid is responsive from ~320px → ~1440px.
- Status and priority badges visually distinct and readable.
- Hover and keyboard focus states are visible.
- Text contrast passes AA (spot-check with browser devtools contrast checker).

### Accessibility checks

- Tab through the page: focus order is logical, focus rings are visible.
- Status is not conveyed by color alone (badge text is present).
- Landmarks (`header`, `main`) are present so screen readers can navigate.
- Test at 200% browser zoom — no clipped content, no horizontal scroll.

### HTML / CSS sanity

- HTML validates (no unclosed tags, valid attributes).
- CSS has no unknown properties or unclosed blocks.
- No inline styles that override the design system.

---

## Edge cases to handle

- Empty `projects` array → render an empty-state message, not a blank page.
- `fetch('project-data.json')` fails (e.g., served from `file://`) → render an error message telling the user to launch via the VS Code configuration.
- Long project names / long `recentActivity` strings → cards should wrap gracefully, not overflow.
- Unknown `status` or `priority` value → fall back to a neutral badge style rather than an unstyled element.
- Port 5500 already in use → surfaced in the terminal; document in a comment or README note that the port can be changed in `launch.json`.
- Very narrow viewport (320px) → single-column, no horizontal scroll.
- `prefers-reduced-motion` → skip hover/transition animations.

---

## Open questions

- Should the dashboard include any client-side filtering (by status or priority), or is a static render sufficient for this exercise? *Assumption: static render only, per the "lightweight" brief.*
- Is there a preferred port other than 5500? *Assumption: 5500, matching the validation gate.*
- Should we include a "last updated" timestamp per project? *Assumption: no — not in the required field list.*
- Should the launch configuration use `type: "node"` with `runtimeExecutable: "python3"`, or a different debugger type? *Assumption: whichever type keeps the JSON strict and enables `serverReadyAction` — the Coder decides during implementation.*
