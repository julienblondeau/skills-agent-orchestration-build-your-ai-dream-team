# Project Pulse — Final handoff

Project Pulse is a static dashboard for Mona's team. It shows each project's owner, status, recent activity, and priority as a card. It is plain HTML, CSS, and JS with no build step, and it lives in `app/`.

## Agent team

| Agent | Model | What it did in this build |
| --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Coordinated the phases and delegated each task to a specialist with a fixed file scope. It did not implement anything itself. |
| Planner | Claude Opus 5.5 (copilot) | Wrote `docs/project-pulse-plan.md`: file assignments, data schema, CSS class contract, dependencies, and validation expectations. |
| Designer | Gemini 3.7 Flash (copilot) | Wrote `app/styles.css` and the markup guidance (landmarks, heading order, class contract) that the Coder followed. |
| Coder | GPT-5.5 (copilot) | Wrote `app/project-data.json`, `.vscode/launch.json`, and `app/index.html`, and ran validation. |

## Files

| Path | Purpose |
| --- | --- |
| `app/index.html` | Dashboard page. Fetches the data and renders one `.project-card` per project, plus loading, empty, and error states. |
| `app/styles.css` | Visual design: responsive card grid, status and priority badges, focus and reduced-motion styles. |
| `app/project-data.json` | Sample data: a top-level `projects` array with 6 projects. |
| `.vscode/launch.json` | VS Code launch config that serves `app/` on port 5500 and opens the dashboard in the browser. |

## How to run

1. Open the **Run and Debug** view in VS Code.
2. Select **Run Project Pulse Dashboard**.
3. Press **F5**. The browser opens `http://localhost:5500/index.html`.

Fallback (terminal):

```bash
cd app && python3 -m http.server 5500
```

Then open `http://localhost:5500/index.html`.

## Dashboard validation results

Run on 2026-11-18 in the Codespace. Every result below was observed directly.

| Check | Result |
| --- | --- |
| `python3 -m json.tool app/project-data.json` | PASS |
| `python3 -m json.tool .vscode/launch.json` | PASS |
| `project-data.json` has a top-level `projects` key | PASS (6 projects) |
| Every project has `name`, `owner`, `status`, `recentActivity`, `priority` | PASS (no missing fields) |
| `launch.json` has no comments | PASS |
| Launch config name is `Run Project Pulse Dashboard` | PASS |
| `cwd` is `${workspaceFolder}/app` | PASS |
| `command` is `python3 -m http.server 5500` | PASS |
| `serverReadyAction.uriFormat` is `http://localhost:%s/index.html` | PASS |
| `index.html` contains `<title>Project Pulse</title>`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority` | PASS (all 7) |
| `styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `@media` | PASS (all 5) |
| Every class in the HTML and JS has a CSS selector (24 classes) | PASS (none missing; no unused selectors either) |
| Runtime: `python3 -m http.server 5500` from `app/` (port 5500 was free) | PASS |
| `GET /index.html`, `/styles.css`, `/project-data.json` | PASS (HTTP 200 each) |
| `index.html` is the dashboard, not a directory listing | PASS (title "Project Pulse"; no "Directory listing" text) |
| jsdom render against the live server | PASS: 6 `.project-card` elements, all 4 status and 3 priority badge styles shown, summary "2 On track, 2 At risk, 1 Blocked, 1 Done", loading message hidden |
| Edge case: `null` entry inside `projects` | PASS (fixed): `renderProjects` now filters out non-object entries before rendering the summary and cards. jsdom with a fetch stub: `[null, valid]` → 1 card, no error; `[null]` and `[]` → "No projects yet" |

Not checked in this pass: a real browser launch through F5, the visual layout at 375/768/1280px, or a Lighthouse/axe audit.

## Project handoff notes

### Extending the data

- Add an object to the `projects` array in `app/project-data.json` with `name`, `owner`, `status`, `recentActivity`, `priority`, and an optional `summary`.
- `status` must be one of `on-track`, `at-risk`, `blocked`, or `done`. `priority` must be one of `high`, `medium`, or `low`. Any other value shows a neutral "Unknown" badge.
- Run `python3 -m json.tool app/project-data.json` after editing, then reload the page. No build step is needed.

### Known risks

- **`serverReadyAction` with `node-terminal` has not been confirmed in a real browser.** If F5 starts the server but no browser opens, switch the config to `"type": "debugpy"`, `"module": "http.server"`, `"args": ["5500"]` and keep the same `serverReadyAction`. Until then, open `http://localhost:5500/index.html` manually.
- **Port 5500 in use.** The server fails to start if another process holds the port. Stop that process, or run the fallback command on another port.
- **`:has()` support for the card stripe.** The colored top stripe on each card relies on `:has()`. Older browsers show a neutral grey stripe instead. The text badge still shows the status, so no information is lost.
- **`file://` will not load the data.** `fetch` fails when `index.html` is opened directly as a file, and the page then shows an error message. Always open the page through the server.

### Suggested next steps

1. Press F5 in a Codespace to confirm the browser opens automatically. Use the `debugpy` fallback if it does not.
2. Check the layout at 375px, 768px, and 1280px, and run a Lighthouse or axe accessibility audit.
3. Optional: add filtering or sorting by status or priority, and a "last updated" date per project.
