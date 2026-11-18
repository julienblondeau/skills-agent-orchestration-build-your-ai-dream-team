# Project Pulse — Implementation Plan

## 1. Overview / goal

Project Pulse is a small **static web dashboard** for Mona's contributors. It shows the status and health of the team's projects in one polished, card-based view. From the brief (`.github/project-pulse-brief.md`), it should show:

- which projects are active
- who owns each project
- each project's current status
- recent activity
- priority or risk level
- a short, contributor-friendly summary

Constraints:
- Plain HTML, CSS and JS, with no build step or dependencies. It lives in `app/`.
- Data comes from `app/project-data.json`, which has a top-level `projects` array.
- It runs from the VS Code launch configuration **Run Project Pulse Dashboard**. That config serves `app/` with `python3 -m http.server 5500` and opens `http://localhost:%s/index.html`, so the user sees the dashboard and not a directory listing.
- Agents never stage, commit or push. The learner handles git.

## 2. File assignments

| File | Owner agent | Purpose |
| --- | --- | --- |
| `docs/project-pulse-plan.md` | Planner | This plan. |
| `docs/project-pulse-design.md` *(optional, design spec only)* | Designer | Layout, color/token, badge and accessibility spec, plus the agreed CSS class contract. It can go inline in the hand-off instead of a file. |
| `app/index.html` | Coder (follows Designer guidance) | Semantic page structure. Exact title "Project Pulse". Links `styles.css`. Fetches `project-data.json`. Renders one `.project-card` per project with name, owner, status, recentActivity and priority. Includes loading, empty and error states. |
| `app/styles.css` | Designer | Visual design: `.dashboard` layout, `.project-card` (with `border-radius` and `box-shadow`), status and priority badges, typography, spacing, responsive grid, focus styles, reduced-motion handling. |
| `app/project-data.json` | Coder | Strict JSON with a top-level `"projects"` key. Each project has `name`, `owner`, `status`, `recentActivity` and `priority`, plus an optional `summary`. Include 4–6 sample projects. |
| `.vscode/launch.json` | Coder | Strict JSON with no comments. Config named **Run Project Pulse Dashboard**. `cwd` is `${workspaceFolder}/app`. Runs `python3 -m http.server 5500`. A `serverReadyAction` opens `http://localhost:%s/index.html`. |

Files that must not change: `.vscode/tasks.json`, `.github/**`, `.devcontainer/**`, `scripts/**`.

### Shared contract (agree before Phase 2)

**Data schema** (`app/project-data.json`):

    {
      "projects": [
        {
          "name": "string",
          "owner": "string",
          "status": "on-track | at-risk | blocked | done",
          "recentActivity": "string (short sentence, may include a date)",
          "priority": "high | medium | low",
          "summary": "string (optional)"
        }
      ]
    }

**CSS class contract** (Designer defines, Coder uses in markup and JS):
- `.dashboard`: the main container.
- `.dashboard-header`: the page header area.
- `.project-grid`: the grid that holds the cards.
- `.project-card`: one card per project (an `<article>` element).
- `.project-name`, `.project-owner`, `.project-activity`, `.project-summary`: the text fields on a card.
- `.status-badge` plus a modifier: `.status-on-track`, `.status-at-risk`, `.status-blocked` or `.status-done`.
- `.priority-badge` plus a modifier: `.priority-high`, `.priority-medium` or `.priority-low`.
- `.dashboard-message`: shared by the loading, empty and error states.

## 3. Designer responsibilities

- **Owns** `app/styles.css` and the design guidance for the markup in `app/index.html`. Designer does not edit `index.html` directly.
- **Information hierarchy.** Header first: "Project Pulse", a one-line description, and optionally a summary count of projects by status. Then the card grid. On each card, the name stands out most, followed by the status badge, owner, priority, recent activity and summary.
- **Layout.** Use CSS Grid: `repeat(auto-fill, minmax(280px, 1fr))`, giving 1 column on mobile and 2–4 on larger screens. Center the content with a sensible max-width and keep spacing consistent.
- **Visual design.** Cards have `border-radius` and `box-shadow`. Status and priority badges get distinct colors. Use a clear system font stack and CSS custom properties for colors and spacing. The first view should clearly look like a dashboard, not a plain page.
- **Accessibility.**
  - Text contrast at least 4.5:1, and badges at least 3:1 for non-text contrast. Badges must not rely on color alone; they always show a text label.
  - Visible `:focus-visible` styles.
  - Respect `prefers-reduced-motion`.
  - Recommend landmarks for the markup: `<header>`, `<main>`, `<article>`, and a heading order of h1, then h2 per card.
- **Deliverable.** The CSS class contract above, then `app/styles.css`, plus a note to Coder on the markup structure.

## 4. Coder responsibilities

- **`app/project-data.json`**: write valid JSON that follows the schema. Use a variety of statuses and priorities so every badge style gets shown.
- **`app/index.html`**:
  - `<!DOCTYPE html>`, `lang="en"`, `meta charset`, `meta viewport`, `<title>Project Pulse</title>`, an `<h1>` reading Project Pulse, and `<link rel="stylesheet" href="styles.css">`.
  - Semantic landmarks, with the `.dashboard` container inside `<main>`.
  - An inline `<script>` that runs `fetch('project-data.json')`, checks `response.ok`, and parses the JSON. For each entry in `data.projects` it builds an `<article class="project-card">` showing `name`, `owner`, `status`, `recentActivity` and `priority`, using the class contract.
  - Set text with `textContent`, not `innerHTML`.
  - Map status and priority values to modifier classes, with a fallback for unknown values.
  - Show loading, empty ("No projects yet") and error messages in `.dashboard-message`. Use `role="status"` or `aria-live="polite"` on it.
- **`.vscode/launch.json`**: strict JSON with no comments. Suggested shape:

      {
        "version": "0.2.0",
        "configurations": [
          {
            "name": "Run Project Pulse Dashboard",
            "type": "node-terminal",
            "request": "launch",
            "command": "python3 -m http.server 5500",
            "cwd": "${workspaceFolder}/app",
            "serverReadyAction": {
              "pattern": "Serving HTTP on .* port ([0-9]+)",
              "uriFormat": "http://localhost:%s/index.html",
              "action": "openExternally"
            }
          }
        ]
      }

- Report to Orchestrator: the files changed, the validation results, and any remaining risks.

## 5. Dependencies

| Task | Depends on | Why |
| --- | --- | --- |
| T1 Data schema + `app/project-data.json` (Coder) | This plan | The schema is fixed above. |
| T2 Design direction + CSS class contract (Designer) | This plan | No other dependency. |
| T3 `.vscode/launch.json` (Coder) | This plan | Only needs the folder layout (`app/index.html`), not the file's contents. |
| T4 `app/index.html` markup + rendering (Coder) | **T1, T2** | Rendering needs the data field names (T1) and the class names and structure (T2). |
| T5 `app/styles.css` (Designer) | **T2**, ideally **T4** | Styles must target the agreed classes. Checking against the real markup avoids mismatches. |
| T6 Integrated validation (Orchestrator → Coder/Designer) | **T1–T5** | It tests the whole app. |

Dependency graph: `T1 + T2 → T4 → T5 → T6`. `T3` can run any time before `T6`.

## 6. Parallel work decisions

**Parallel (Phase 1):**
- T1 (data file) and T3 (launch.json) can run alongside T2 (design direction). The files don't overlap: Coder owns `app/project-data.json` and `.vscode/launch.json`, Designer owns the design spec. None of these needs the others' output.
- T1 and T3 are both Coder's, so Coder can do them in one task.

**Sequential:**
- **T4 after T1 and T2.** `index.html` uses both the data field names and the CSS class names.
- **T5 after T4 (recommended).** Styling the real DOM avoids unused or missing selectors. If time matters, T5 can run alongside T4 because the class contract is fixed and the files are different. In that case, T6 must check that the selectors match.
- **T6 last.** It needs every file in place.
- Two agents never edit the same file in the same phase. `index.html` belongs only to Coder and `styles.css` only to Designer.

| Phase | Tasks | Mode |
| --- | --- | --- |
| 1 | T1 + T3 (Coder) ‖ T2 (Designer) | Parallel |
| 2 | T4 (Coder) | Sequential |
| 3 | T5 (Designer) | Sequential (or parallel with T4 if the contract is fixed) |
| 4 | T6 validation | Sequential |

## 7. Validation expectations

**Static checks** (the Step 3 workflow checks the same things):
- `python3 -m json.tool app/project-data.json` succeeds. The file has a top-level `projects` key, and every project has `name`, `owner`, `status`, `recentActivity` and `priority`.
- `python3 -m json.tool .vscode/launch.json` succeeds. The file contains `Run Project Pulse Dashboard` and `index.html`, its `cwd` is `${workspaceFolder}/app`, and it has no comments or trailing commas.
- `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
- `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
- Every class used in the markup and JS has a matching selector in the CSS, and the reverse.

**Runtime checks:**
- Run **Run Project Pulse Dashboard**. The browser opens `/index.html` (not a directory listing) and shows one card per project, with no console errors and no 404s.
- Fallback check: `cd app && python3 -m http.server 5500`, then `curl -s localhost:5500/index.html` and `curl -s localhost:5500/project-data.json` both return 200.
- Error state: temporarily break or rename the data file. The page shows a readable error message instead of a blank page. Then restore the file.

**Responsive and accessibility checks:**
- Layout works at about 375px (1 column), 768px and 1280px or wider, with no horizontal scrolling.
- Landmarks present (`header` and `main`), one `h1`, and a logical heading order.
- Contrast is at least 4.5:1 for text, and badges have text labels.
- `:focus-visible` is styled, and `prefers-reduced-motion` is respected.
- Optional: run a Lighthouse or axe check in the browser devtools.

## 8. Edge cases

- **Opening via `file://`.** `fetch` fails over `file://`, so the page must be opened through the launch config or the local server. The error message should hint at this.
- **Empty `projects` array, or a missing `projects` key.** Show the empty or error state.
- **Missing fields on a project.** Show a placeholder ("—"); don't crash.
- **Unknown status or priority values.** Use a neutral badge class.
- **Untrusted text, or HTML inside data strings.** Use `textContent`.
- **Long names or long activity text.** Text wraps; cards don't overflow.
- **Port 5500 already in use.** The launch fails. Document this as a risk and ask the learner to stop other servers.

## 9. Open questions / risks

- **Does `serverReadyAction` open the browser with `"type": "node-terminal"`?** This needs a real check in a Codespace. If it doesn't open, switch to a Python-extension launch type (`"type": "debugpy"`, `"module": "http.server"`, `"args": ["5500"]`), keeping the same `serverReadyAction`.
- **Should a separate `docs/project-pulse-design.md` be created?** Default: no. The design spec goes inline in the hand-off to Coder.
- **Is a header summary (counts by status) wanted?** Recommended, but optional.
