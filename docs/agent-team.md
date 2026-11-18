# Agent team

This team of custom agents builds Mona's Project Pulse dashboard. GitHub Copilot CLI in a Codespace orchestrates the work.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Breaks down the request, delegates to specialists, enforces file scopes and phase order, and reports the final outcome. It does not implement work itself. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repository and docs, then produces an implementation plan with file assignments, dependencies, and edge cases. It does not write code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Writes code, fixes bugs, and implements logic within its assigned files. For Project Pulse it also creates `.vscode/launch.json` so the dashboard opens in a browser. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Owns UI/UX and visual design, including the dashboard layout, project cards, status badges, and responsive styling. | `.github/agents/designer.agent.md` |

All agents leave git operations to the learner. None of them stages, commits, or pushes changes.
