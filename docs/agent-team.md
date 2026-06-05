# Agent team

This project uses four custom agents defined under `.github/agents/` to build Mona's Project Pulse dashboard. Orchestration is performed interactively through **GitHub Copilot CLI running inside a Codespace**.

| Agent | Model | Definition | Responsibility |
|---|---|---|---|
| **Orchestrator** | Claude Opus 4.7 | `.github/agents/orchestrator.agent.md` | Coordinates the other agents. Breaks requests into phases, delegates tasks with explicit file scopes, runs independent work in parallel, and reports the final outcome. Does not write code. |
| **Planner** | Claude Opus 4.7 | `.github/agents/planner.agent.md` | Researches the codebase and dependencies, identifies edge cases and risks, and produces an ordered implementation plan with file assignments and parallelism guidance for the Orchestrator. Does not write code. |
| **Coder** | GPT-5.5 | `.github/agents/coder.agent.md` | Implements features, fixes bugs, and writes logic within the file scope assigned by the Orchestrator. Also creates support configuration (e.g. `.vscode/launch.json`) needed to run the app. |
| **Designer** | Gemini 3.1 Pro | `.github/agents/designer.agent.md` | Handles UI/UX, accessibility, visual styling, and responsive layout. For Project Pulse this means polished dashboard cards, status badges, priority treatment, and deterministic CSS hooks such as `.dashboard` and `.project-card`. |

> **Orchestration note:** All agent coordination happens through GitHub Copilot CLI in a Codespace. The learner drives every git operation (stage, commit, push) manually — no agent commits or pushes changes on their own.
