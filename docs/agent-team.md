# Agent team for Mona's Project Pulse dashboard

This project uses a small custom agent team defined under the repository's agent folder and coordinated through GitHub Copilot CLI in a Codespace.

## Team overview

### 1. Orchestrator
- Model: Claude Opus 4.7 (copilot)
- Responsibility: Breaks the dashboard work into phases, delegates tasks to specialist agents, and verifies that the pieces fit together without implementing the work directly.
- Definition: .github/agents/orchestrator.agent.md

### 2. Planner
- Model: Claude Opus 4.7 (copilot)
- Responsibility: Researches the repo, reads the relevant files, identifies edge cases and dependencies, and produces a practical implementation plan with file ownership, sequencing, validation expectations, and open questions.
- Definition: .github/agents/planner.agent.md

### 3. Coder
- Model: GPT-5.5 (copilot)
- Responsibility: Implements the actual code changes, handles logic and bug fixes, and can create support files for runnable app work such as launch configuration for the dashboard preview.
- Definition: .github/agents/coder.agent.md

### 4. Designer
- Model: Gemini 3.1 Pro (copilot)
- Responsibility: Focuses on the user experience, accessibility, layout, visual hierarchy, interaction flow, and polished dashboard styling for Project Pulse.
- Definition: .github/agents/designer.agent.md

## How the team works together

The Orchestrator uses GitHub Copilot CLI in the Codespace to coordinate the workflow: it asks the Planner for a structured plan, then delegates implementation across the Coder and Designer based on file scope and dependencies. The Coder handles the application logic, while the Designer shapes the interface so the dashboard looks like a polished Project Pulse experience.

This configuration keeps the work clear, parallelizable where appropriate, and aligned with the repository's expectations for staged, reviewable progress.