# GitHub Copilot Setup

This repository is a GitHub Copilot workspace setup for a structured multi-agent development workflow.

The repository is intended to serve as a reusable starting point for projects that want:

- a single orchestration entry point
- role-based agents for architecture, development, and QA
- shared skills and review standards
- predictable documentation and handoff conventions

## What This Repo Provides

The setup is organized around a Copilot agent system with a clear separation of responsibilities:

- `project-manager` is the main entry point for work
- architecture, development, and QA responsibilities are split across dedicated agents
- reusable skills define standards for coding, testing, architecture, and review
- instructions enforce workflow checkpoints, including documentation review gates

## Architecture Baseline

This setup assumes a deliberately simpler architecture baseline:

- frontend work uses `Feature-Sliced Design (FSD)`
- backend work uses `Clean Architecture`
- `DDD` and `Hexagonal Architecture` are optional reference skills, not default workflow requirements

The primary conventions live in:

- `AGENTS.md`
- `.github/copilot-instructions.md`
- `.github/agents/`
- `.github/skills/`
- `.github/instructions/`

## Repository Structure

```
AGENTS.md
.github/
├── copilot-instructions.md
├── agents/
├── docs/
├── instructions/
└── skills/
```

## Agent Workflow

The intended workflow starts with the Project Manager agent.

From there, the system is designed to:

1. clarify scope and constraints
2. produce architecture guidance when needed
3. break the work into tracked tasks
4. delegate implementation to development agents
5. validate outcomes through QA agents
6. record outputs under `.github/docs/`

## Key Files

### `AGENTS.md`

Documents the agent roster, delegation model, workflow, evidence expectations, and documentation structure.

### `.github/copilot-instructions.md`

Defines the global project context, including stack assumptions, architectural rules, and coding conventions.

### `.github/agents/`

Contains the role definitions for each agent, including responsibilities and handoff expectations.

### `.github/skills/`

Contains the reusable standards each agent is expected to load and follow for specific tasks such as:

- feature-sliced design for frontend
- clean architecture for backend
- frontend and backend patterns
- testing
- code review
- security and performance review

### `.github/instructions/`

Contains targeted instructions that apply to specific file patterns or workflow checkpoints.

One important rule already included in this repo: any markdown written under `.github/docs/` must be reviewed and explicitly approved by user before the workflow continues.

## How To Use This Setup

1. Open this folder in an IDE with GitHub Copilot enabled.
2. Keep the `AGENTS.md` file and `.github/` folder at the workspace root so Copilot can discover the instructions, agents, and skills.
3. Start work through the Project Manager agent rather than asking individual implementation agents directly.
4. Review generated documentation under `.github/docs/` whenever the workflow pauses for approval.

## Customizing The System

You can adapt the setup by editing:

- `AGENTS.md` to change the agent roster or workflow
- `.github/copilot-instructions.md` for project-specific context and rules
- `.github/agents/*.agent.md` for agent behavior and responsibilities
- `.github/skills/*/SKILL.md` for reusable standards and workflows
- `.github/instructions/*.instructions.md` for path-targeted rules and gates
