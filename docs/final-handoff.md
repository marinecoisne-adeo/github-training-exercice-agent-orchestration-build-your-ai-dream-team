# Project Pulse final handoff

Project Pulse is a static HTML/CSS dashboard for Mona's team. It reads `app/project-data.json` and shows each project's status, priority and recent activity on project cards. The plan is in `docs/project-pulse-plan.md` and the team is described in `docs/agent-team.md`.

## Agent team

| Agent | Model | What it delivered in this project |
| --- | --- | --- |
| Orchestrator | Claude Opus 5.5 | Split the request into phases (plan → parallel build → integration review → validation → handoff), assigned file scopes, and routed work between agents. It wrote no implementation code. |
| Planner | Claude Opus 5.5 | Wrote `docs/project-pulse-plan.md`: scope, file ownership, shared JSON and markup contracts, phases, validation table V1–V12, and risks. |
| Designer | Gemini 3.7 Flash | Wrote `app/styles.css` (layout, cards, badges, responsive grid, focus and reduced-motion styles) and reviewed it against the markup `app/index.html` actually renders. |
| Coder | GPT-5.5 | Wrote `app/index.html`, `app/project-data.json` and `.vscode/launch.json`, and ran the command-based validation below. |

## Deliverables

- `app/index.html`: page structure plus an inline script that fetches `project-data.json` and renders `.project-card` elements, summary chips, and loading, empty and error states.
- `app/styles.css`: all visual design, including the `.dashboard` layout, `.project-card` styling (`border-radius`, `box-shadow`), status and priority badges, and `@media` rules.
- `app/project-data.json`: 6 sample projects under a top-level `projects` array. Every status and priority value appears at least once.
- `.vscode/launch.json`: the **Run Project Pulse Dashboard** launch configuration, in strict JSON.

## How to run

1. Open **Run and Debug** in VS Code (Ctrl+Shift+D / Cmd+Shift+D).
2. Select **Run Project Pulse Dashboard**, which is defined in `.vscode/launch.json`, and press **Start**.
3. The configuration runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`, so it serves `app/` on port 5500.
4. When the terminal prints `Serving HTTP on … port 5500`, `serverReadyAction` opens `http://localhost:%s/index.html` (`%s` becomes `5500`). You should see the dashboard, not a directory listing.
5. Stop the server with the Stop button or Ctrl+C in the terminal.

**If the browser does not open**, use the **Ports** panel to open forwarded port 5500, then go to `/index.html`. Or run it yourself:

```bash
cd app && python3 -m http.server 5500
```

**Note:** opening `index.html` directly (`file://`) will not work, because the browser blocks `fetch()`. The page shows an error that explains this.

## Dashboard validation results

The Coder ran these checks in the Codespace on 2026-11-16.

| # | Check | How | Result |
| --- | --- | --- | --- |
| V1 | Data JSON is valid | `python3 -m json.tool app/project-data.json` | ✅ Pass |
| V2 | Schema is complete | Python: `projects` is a non-empty list (6 items). All 5 required fields are non-empty strings. Status and priority values come from the allowed sets. | ✅ Pass. Statuses: on-track 2, at-risk 2, blocked 1, completed 1. Priorities: high 2, medium 2, low 2. Every value appears at least once. |
| V3 | Launch JSON is correct | `python3 -m json.tool .vscode/launch.json`, then a Python check of the name, command, `cwd` and `uriFormat`; also checked for comments | ✅ Pass. No comments. `type` is `node-terminal`. |
| V4 | Required literals are present | Python substring check. HTML: `<title>Project Pulse</title>`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`. CSS: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `@media` | ✅ Pass (all present) |
| V4b | CSS braces are balanced | Count `{` and `}` in `app/styles.css` | ✅ Pass (74 / 74) |
| V4c | No inline styles in HTML | Searched for `style=` and `<style` | ✅ Pass (none) |
| V5 | Files are served over HTTP | `python3 -m http.server 5500` from `app/`, then `curl` | ✅ Pass. `/index.html`, `/styles.css` and `/project-data.json` all return 200. `GET /` returns the dashboard: it contains "Project Pulse" and no "Directory listing". |
| V5b | Server stopped | Stopped the server, then checked port 5500 with `ss -ltn` | ✅ Pass (port free) |
| V6/V7 | Cards render (simulated) | Ran the page's inline script in Node 22 with a minimal fake DOM and real `fetch` against the local server | ✅ Pass. 6 `.project-card` elements. Badge classes match the JSON (for example `status-badge--at-risk`, `priority-badge--high`). `.project-activity` shows the `recentActivity` text. Summary chips add up to 6. Message: "Showing 6 projects (5 active)." |
| V7b | Unknown values fall back | Fake DOM with `status: "paused"` and `priority: "urgent"` | ✅ Pass. `status-badge--unknown`, `priority-badge--low`, and an "Unknown" summary chip. |
| V8 | Empty and error states | Fake DOM with `projects: []`, an HTTP 404, and a `file:` protocol fetch failure | ✅ Pass. The empty state shows "No projects to show yet." Both errors add `status-message--error`. The `file://` error explains that fetch is blocked and gives the run instructions. |
| V12 | Scope check | `git status --porcelain` before writing this file; `git log --oneline -5` | ✅ Pass. The working tree was clean; app files are committed in `e811da9`. |

No headless browser was available, so no real browser rendering was run. The fake-DOM check covers the script logic only, not CSS or layout.

**Still needs a human in a real browser (V6, V9, V10, V11):**

- Launch through **Run Project Pulse Dashboard** and confirm the browser opens `/index.html` (V11).
- Check that the DevTools console shows no errors.
- Look at the layout at 375px (one column, no horizontal scroll), 768px (two columns) and 1280px (three or more columns), and at 200% zoom.
- Check colour contrast with a contrast tool or axe DevTools (badges need at least 4.5:1).
- Check keyboard use: Tab order is logical, `:focus-visible` is visible, and there are no focus traps.

## Learner handoff checklist

- [ ] Run **Run Project Pulse Dashboard** from Run and Debug and confirm the dashboard (not a directory listing) opens on port 5500.
- [ ] Confirm 6 project cards appear and the summary shows On track 2, At risk 2, Blocked 1, Completed 1.
- [ ] Open DevTools and confirm the console has no errors.
- [ ] Use responsive mode to check 375px, 768px and 1280px, then check 200% zoom.
- [ ] Run axe DevTools or a contrast checker, and Tab through the page.
- [ ] Stop the debug session so port 5500 is free.
- [ ] Review the diff, then use Copilot CLI to stage, commit and push `docs/final-handoff.md`.

## Known risks and open questions

**Risks (from plan §9):**

- **`file://`:** `fetch()` is blocked, so always serve over HTTP. The error message says this.
- **Port 5500 busy:** the server will not start. Stop other servers first; the port is fixed by the exercise.
- **`serverReadyAction` does not fire:** open port 5500 from the Ports panel instead.
- **Codespaces forwarding / pop-ups:** `openExternally` depends on automatic port forwarding. You may need to allow pop-ups.
- **Contract drift:** a misspelled class leaves cards unstyled. Any contract change must go through the Orchestrator.
- **Designer cannot run commands:** contrast and responsive behaviour were reviewed only by reading the CSS. They still need the visual checks above.
- **XSS:** the script uses `textContent` only. Keep it that way if you extend it.

**Open questions:**

1. Should `recentActivity` stay a free-text string, or become a structured `{ date, description }` object?
2. Should `completed` projects stay on the dashboard? They are currently shown with neutral styling.
3. Is the summary strip required, or only nice to have? It is currently implemented.
4. Is `node-terminal` acceptable as the launch `type`? The alternative, `debugpy`, would need the Python extension.
