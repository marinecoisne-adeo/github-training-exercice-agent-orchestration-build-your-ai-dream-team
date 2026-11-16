# Project Pulse implementation plan

## 1. Goal and scope

**Goal:** Build Mona's **Project Pulse** dashboard. It is a small, polished, static HTML/CSS dashboard that contributors use to see the state of the team's projects at a glance. One JSON file holds the project data, and the page renders it into project cards on the client side.

**The dashboard shows:**

- A clear page title, exactly `Project Pulse`, plus a short tagline.
- A summary strip with the number of projects in each status, so active and finished work is easy to tell apart.
- A responsive grid of **project cards** (`.project-card`). Each card shows:
  - project `name`
  - `owner`
  - `status` as a text badge with colour semantics
  - `priority` (risk level) as a text badge
  - `recentActivity`
  - optional `summary`, a short note written for contributors
- Visible loading, empty and error states.
- A VS Code launch configuration, **Run Project Pulse Dashboard**. It serves `app/` and opens `index.html`, not a directory listing.

**Out of scope:**

- Build tools, frameworks, npm packages, bundlers or extra JS/CSS files. Only the three app files are allowed.
- A backend, APIs, authentication, persistence or editing data from the UI.
- Live data from GitHub or other sources. The data is static sample JSON.
- Filtering, sorting, search, charts and dark-mode toggles. These could be added later.
- Changes to `.vscode/tasks.json`, `.devcontainer/*`, workflows, the agent definitions or `docs/agent-team.md`.
- Git operations. The learner stages, commits and pushes.

**Constraints from the exercise:** see `.github/project-pulse-brief.md`, `.github/steps/3-step.md` and `.github/workflows/3-step.yml`.

- `app/index.html` must contain these literal strings: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
- `app/styles.css` must contain `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
- `app/project-data.json` must be valid JSON with a top-level `projects` key. Every project needs `name`, `owner`, `status`, `recentActivity` and `priority`.
- `.vscode/launch.json` must be strict JSON with no comments. It needs a configuration named `Run Project Pulse Dashboard` that uses `python3 -m http.server 5500`, runs with `cwd` set to `${workspaceFolder}/app`, and has a `serverReadyAction` that opens `http://localhost:%s/index.html`.

## 2. File assignments

Each file has exactly one owner. No agent edits a file it does not own.

| File | Owner | Purpose |
| --- | --- | --- |
| `app/project-data.json` | **Coder** | Sample data under a top-level `projects` array, following the schema in §6. |
| `app/index.html` | **Coder** | Page structure, semantic landmarks, link to `styles.css`, and an inline `<script>` that fetches `project-data.json` and builds `.project-card` elements using the markup contract in §6. |
| `.vscode/launch.json` | **Coder** | The **Run Project Pulse Dashboard** configuration. Strict JSON, no comments. It serves `app/` on port 5500 and opens `index.html`. |
| `app/styles.css` | **Designer** | All visual design: layout, typography, card and badge styling, status and priority colours, responsive grid, focus styles and reduced motion. |
| `docs/project-pulse-plan.md` | Planner (via Orchestrator) | This plan. Read-only for Designer and Coder. |

Notes:

- The Designer cannot run commands. Its work must be reviewable by reading the files.
- The Coder runs every command-based check.
- Do not add inline `style` attributes or `<style>` blocks to `app/index.html`. All styling goes in `app/styles.css`, so the two scopes stay separate.

## 3. Designer responsibilities

**May touch:** `app/styles.css` only. The Designer may read every other file.

Deliverables:

1. **Layout**
   - `.dashboard` is the main container: centred, with a max-width of about 1200px and comfortable padding.
   - `.site-header` holds the title and tagline.
   - `.pulse-summary` is a row of stat chips.
   - `.project-grid` uses CSS Grid, for example `repeat(auto-fill, minmax(280px, 1fr))`, with consistent gaps.
2. **Visual style**
   - Project cards look polished: `border-radius` around 12px, a soft `box-shadow`, a white or near-white surface on a light neutral page background, and clear spacing inside each card.
   - Typography uses a system font stack and a clear hierarchy: `h1` > section `h2` > card `h3`, then metadata.
   - Define CSS custom properties in `:root` for colours, spacing, radius and shadow.
3. **Status and priority semantics.** Badges always show text, so colour is never the only signal.
   - Status:
     - `on-track`: green
     - `at-risk`: amber
     - `blocked`: red
     - `completed`: blue or grey
     - `unknown`: neutral grey
   - Priority:
     - `high`: strong red or orange outline or fill
     - `medium`: amber
     - `low`: neutral or green
   - Optionally add a coloured left border on `.project-card` using `[data-status="…"]`.
4. **Accessibility**
   - Text and badge contrast is at least 4.5:1, or 3:1 for large text and UI boundaries.
   - Focus styles are visible through `:focus-visible`.
   - A `.visually-hidden` utility class is available.
   - `@media (prefers-reduced-motion: reduce)` turns off any hover or transition animation.
   - The layout must not break at 200% zoom.
5. **Responsive behaviour**
   - Single column at 375px or less with no horizontal scroll.
   - Two or more columns at tablet widths, and three or more at 1280px or more.
   - The summary chips wrap.
6. **State styling:** `.status-message` for neutral and loading messages, `.status-message--error` for errors, and `.empty-state`.
7. **Report:** design decisions, contrast values chosen for each badge, which contract class names were styled, and any markup change requests. Markup changes go through the Orchestrator to the Coder. The Designer never edits HTML.

## 4. Coder responsibilities

**May touch:** `app/project-data.json`, `app/index.html`, `.vscode/launch.json`.

Deliverables:

1. **Data (`app/project-data.json`)**
   - 5–6 projects that cover every `status` value and every `priority` value in §6.
   - Realistic owners and activity text, plus a `summary` for each project.
   - Valid JSON, UTF-8, with no trailing commas.
2. **HTML structure (`app/index.html`)**
   - Starts with `<!DOCTYPE html>` and uses `lang="en"`.
   - Includes the `<meta charset>` and viewport meta tags.
   - `<title>Project Pulse</title>`.
   - `<link rel="stylesheet" href="styles.css">`.
   - Landmarks `header`, `main.dashboard` and `footer`, following the markup contract in §6.
3. **Rendering logic** (inline `<script>` at the end of `body`, or `defer`)
   - Call `fetch('project-data.json', { cache: 'no-store' })`.
   - Check `response.ok`. If it fails, throw an explicit error that includes the HTTP status.
   - Check that `data.projects` is an array.
   - Build each card with `document.createElement` and `textContent`. Do not put data into `innerHTML`.
   - Map `status` and `priority` to modifier classes, as described in §6.
   - Fill in `.pulse-summary` counts.
   - Show the loading, empty and error states. The error message must explain that the page needs to be served over HTTP. Opening it from `file://` blocks `fetch()`.
   - The literal identifiers `status`, `recentActivity` and `priority` must appear in the script.
4. **Launch configuration (`.vscode/launch.json`)**
   - Strict JSON with `"version": "0.2.0"` and one configuration:
     - `"name": "Run Project Pulse Dashboard"`
     - `"type": "node-terminal"` (built into VS Code's JS debugger, so no extra extension is needed)
     - `"request": "launch"`
     - `"command": "python3 -m http.server 5500"`
     - `"cwd": "${workspaceFolder}/app"`
     - `"serverReadyAction"` containing:
       - `"pattern": "Serving HTTP on .* port ([0-9]+)"`
       - `"uriFormat": "http://localhost:%s/index.html"`
       - `"action": "openExternally"`
   - Do not modify `.vscode/tasks.json`.
5. **Report:** files changed, the output of each validation command (§8), and any remaining risks.

## 5. Dependencies

| # | Task | Owner | Depends on |
| --- | --- | --- | --- |
| D1 | Shared contract: JSON schema and markup/class contract (§6, this plan) | Planner | — |
| D2 | `app/project-data.json` sample data | Coder | D1 (schema) |
| D3 | `app/index.html` structure and rendering logic | Coder | D1 (contract); D2 for end-to-end testing |
| D4 | `.vscode/launch.json` | Coder | None. Needs `app/index.html` (D3) only for the end-to-end launch check. |
| D5 | `app/styles.css` | Designer | D1 (class names, data attributes, modifier values) |
| D6 | Integration review: CSS matches the markup that is actually rendered | Designer (read-only review) and Orchestrator | D3, D5 |
| D7 | Command-based validation (§8) | Coder | D2, D3, D4, D5 |
| D8 | Manual launch and visual check in the Codespace | Learner, guided by the Orchestrator | D7 |

## 6. Parallel work decisions

**Decision:** Run the Coder track (D2 → D3 → D4) and the Designer track (D5) **in parallel** in Phase 2.

**Why this is safe:**

- The file scopes do not overlap. The Coder owns the HTML, JSON and launch files. The Designer owns only the CSS.
- The only coupling between the tracks is the shared contract below. It is fixed before either track starts.

**What must be sequential:**

- Inside the Coder track, the data must exist before rendering can be tested end to end (D2 before the D3 test). The Coder handles this order alone.
- Integration review (D6) and validation (D7) run after both tracks finish.
- Fixes from review run one after another, each routed to the owner of the file.
- If either agent needs to change the contract, the Orchestrator pauses parallel work, updates both agents, and then resumes.

### Shared contract A: JSON schema (`app/project-data.json`)

```json
{
  "lastUpdated": "YYYY-MM-DD (optional string)",
  "projects": [
    {
      "name": "string, required",
      "owner": "string, required",
      "status": "on-track | at-risk | blocked | completed (required)",
      "recentActivity": "string, required (short sentence, e.g. 'Merged PR #42 – 2 days ago')",
      "priority": "high | medium | low (required)",
      "summary": "string, optional (one or two contributor-friendly sentences)"
    }
  ]
}
```

- Status and priority values are lowercase kebab-case.
- An unrecognised `status` renders as `unknown`, and an unrecognised `priority` renders as `low`. In both cases the badge still shows the raw text.
- **Active projects** are those whose `status` is anything except `completed`.

### Shared contract B: markup, class names and IDs (rendered by `app/index.html`, styled by `app/styles.css`)

| Element | Contract |
| --- | --- |
| Page header | `<header class="site-header">` containing `<h1 class="site-title">Project Pulse</h1>` and `<p class="site-tagline">` |
| Main container | `<main id="dashboard" class="dashboard">` |
| Summary | `<section class="pulse-summary" aria-label="Project status summary">` holding `<div class="summary-chip summary-chip--{status}">` items. Each chip has `<span class="summary-count">` and `<span class="summary-label">`. |
| Status message | `<p id="status-message" class="status-message" role="status" aria-live="polite">`. Add `status-message--error` on failure. |
| Projects section | `<section class="projects" aria-labelledby="projects-heading">` with `<h2 id="projects-heading">Projects</h2>` |
| Grid | `<ul id="project-list" class="project-grid" role="list">` |
| Card | `<li class="project-card" data-status="{status}" data-priority="{priority}">`, which wraps `<article class="project-card__body">` |
| Card heading | `<h3 class="project-name">` |
| Badges row | `<div class="project-badges">` containing `<span class="status-badge status-badge--{status}">` and `<span class="priority-badge priority-badge--{priority}">`. Badge text is readable, for example "At risk" or "High priority". |
| Metadata | `<dl class="project-meta">` with `dt`/`dd` pairs for **Owner** (`dd.project-owner`) and **Recent activity** (`dd.project-activity`) |
| Summary text | `<p class="project-summary">`, left out when `summary` is missing |
| Empty state | `<p class="empty-state">No projects to show yet.</p>` inside `.projects` |
| Footer | `<footer class="site-footer">` with a note on the data source and `lastUpdated` |
| Utility | `.visually-hidden` |

Allowed `{status}` values are `on-track`, `at-risk`, `blocked`, `completed` and `unknown`. Allowed `{priority}` values are `high`, `medium` and `low`.

## 7. Phases

1. **Phase 1, plan (done):** Planner produces this document. The Orchestrator saves it to `docs/project-pulse-plan.md`.
2. **Phase 2, build (parallel):**
   - **Coder:** `app/project-data.json` → `app/index.html` → `.vscode/launch.json`, all following contracts A and B.
   - **Designer:** `app/styles.css`, following contract B and the §3 deliverables.
3. **Phase 3, integration review (sequential):**
   - The Designer reads the final `app/index.html` and confirms that every contract class is styled and that no classes are missing or misspelled. Any fixes go only into `app/styles.css`.
   - The Orchestrator checks consistency across all files.
4. **Phase 4, validation (sequential):** the Coder runs every command-based check in §8 and reports the results. Failures go back to the owner of the affected file, one at a time.
5. **Phase 5, handoff:**
   - The learner runs **Run Project Pulse Dashboard** from Run and Debug, checks the UI visually at mobile and desktop widths, and then stops the server.
   - The learner then handles git through Copilot CLI.

## 8. Validation expectations

| # | Check | How | Validator |
| --- | --- | --- | --- |
| V1 | Data JSON is valid | `python3 -m json.tool app/project-data.json` exits 0 | Coder |
| V2 | Schema is complete | A Python one-liner confirms `projects` is a non-empty list, every item has all 5 required fields as non-empty strings, and status and priority values come from the allowed sets | Coder |
| V3 | Launch JSON is valid and correct | `python3 -m json.tool .vscode/launch.json`, then confirm the `name`, `command`, `cwd` (`${workspaceFolder}/app`) and `uriFormat` (`http://localhost:%s/index.html`) | Coder |
| V4 | Required literals for the exercise workflow are present | `grep` for the strings listed in §1 in each file | Coder |
| V5 | Files are served over HTTP | Run `python3 -m http.server 5500` from `app/` in the background. `curl -s -o /dev/null -w '%{http_code}'` returns 200 for `/index.html`, `/styles.css` and `/project-data.json`. Then stop the server. Note: `fetch()` fails under `file://`, so the page must always be served. | Coder |
| V6 | No console errors; every project renders | Open `http://localhost:5500/index.html` in the browser through the launch config. DevTools console shows no errors. The number of cards equals `projects.length`, and the summary counts add up to the total. If a headless browser happens to be available in the container, the Coder may automate this; otherwise it is manual. | Learner, guided by the Orchestrator (Coder if headless is available) |
| V7 | Status and priority indicators are correct | Each card's badge text and modifier class match its JSON values, every status and priority appears at least once, and an unknown status shows a neutral badge | Coder (code review plus V6), Designer (colour mapping) |
| V8 | Error and empty states work | Temporarily point the fetch at a missing file or open the page via `file://`: the error message is visible and readable. With an empty `projects` array, the empty state appears. Revert any temporary changes afterwards. | Coder |
| V9 | Accessibility | Landmarks (`header`, `main`, `footer`); one `h1`; logical heading order; badges carry text, so meaning never depends on colour alone; contrast of at least 4.5:1 (document the values); visible `:focus-visible`; `aria-live` status region; keyboard Tab order is logical with no traps; any decorative icons use `aria-hidden="true"` or `alt=""` | Designer (contrast, focus, colour semantics), Coder (semantics and ARIA) |
| V10 | Responsive layout | At 375px: one column, no horizontal scroll. At 768px: two columns. At 1280px or more: three or more columns. Check at 200% zoom too. | Designer (CSS review), Learner (visual check in the browser) |
| V11 | Launch works in the Codespace | Run and Debug → **Run Project Pulse Dashboard**. The terminal prints "Serving HTTP on … port 5500", port 5500 is forwarded, the browser opens `/index.html` and shows the dashboard, not a directory listing. Then stop the server. | Learner, coordinated by the Orchestrator |
| V12 | Scope check | Only the four assigned files were created or changed, and `tasks.json` is untouched | Orchestrator |

## 9. Risks and open questions

**Risks:**

- **Opening via `file://`:** `fetch()` is blocked. The page must always be served. The error message must say this clearly (V8).
- **Port 5500 already in use:** the server fails to start. Stop any other server first. The port is fixed by the exercise, so do not change it.
- **`serverReadyAction` does not fire:** this can happen if Python's startup line is buffered or worded differently. `node-terminal` runs in a real terminal, so output is line-buffered. If it still does not trigger, open the forwarded port 5500 from the Ports panel and report the issue.
- **Codespaces forwarding:** `openExternally` on `localhost` depends on VS Code automatically forwarding the port. A browser pop-up blocker may need to be allowed.
- **Contract drift:** a misspelled class name leaves cards unstyled. Phase 3 review covers this, and any contract change must go through the Orchestrator.
- **Designer cannot run commands:** contrast and responsive checks are done by reasoning about the CSS and then confirmed visually by the learner.
- **XSS from data:** use `textContent` only.

**Open questions:**

1. Should `recentActivity` be a free-text string, as assumed here, or a structured `{ date, description }` object? A string keeps the exercise keyphrase checks simple.
2. Is a `completed` status wanted, or should the dashboard list active projects only? The plan includes `completed` and shows it with neutral styling.
3. Should the summary strip be required, or only nice to have? The plan includes it, but V6 can treat it as optional if time is short.
4. Is `node-terminal` acceptable as the launch `type`? It needs no extension in the universal image. The alternative is a `debugpy` launch, which would require the Python extension, and that extension is not in `devcontainer.json`.
