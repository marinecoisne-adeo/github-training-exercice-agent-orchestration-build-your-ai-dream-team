# Agent team

This team of custom agents builds Mona's Project Pulse dashboard. I use GitHub Copilot CLI in a Codespace to orchestrate the work.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 | Breaks down requests and delegates to the specialist agents; does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 | Researches the codebase, docs, dependencies and edge cases to produce implementation plans; writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 | Writes code, fixes bugs and implements logic within the assigned file scope. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash | Handles UI/UX, accessibility, information architecture and visual design. | `.github/agents/designer.agent.md` |

## How the team works together

1. **Orchestrator** receives the Project Pulse request from the user in Copilot CLI and breaks it into tasks.
2. **Planner** researches the repository and produces an implementation plan (no code).
3. **Designer** defines the UX, accessibility and visual design of the dashboard.
4. **Coder** implements the plan and the design within the file scope the Orchestrator assigns.
5. **Orchestrator** reviews each result, hands off between agents, and iterates until the dashboard is complete.
