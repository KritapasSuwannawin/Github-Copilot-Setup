---
name: task-breakdown
description: Defines how the project-manager agent breaks down any project goal into a structured tasks.md file. This file is the single source of truth for all work in the project.
---

# Skill: Task Breakdown

## Purpose

This skill defines how the `project-manager` agent breaks down any project goal into a structured `tasks.md` file. This file is the single source of truth for all work in the project.

## When to Use

Invoke this skill before any development begins. No agent should be delegated work until `tasks.md` exists and is complete.

## Process

### Step 1 — Understand the Goal

Restate the user's goal in one sentence. If it's ambiguous, ask clarifying questions before proceeding.

### Step 2 — Identify Epics

Group related work into epics (high-level themes). Typical epics: Auth, Core Feature, UI, API, Infrastructure, Testing.

### Step 3 — Break Into Tasks

Each epic contains tasks. A task must be:

- Independently implementable by one developer in one session
- Testable — has a clear definition of done
- Specific enough that an agent knows exactly what to build

**Full-stack tasks:** If a task requires both frontend and backend work, split it into two linked tasks (e.g. Task 2.1 — API endpoint, Task 2.2 — UI integration) unless the entire change is genuinely trivial. Define the API contract in the backend task's Definition of Done so the frontend task can depend on it.

### Step 4 — Add Subtasks

Each task can have subtasks for fine-grained tracking.

### Step 5 — Declare MCP Inputs & Evidence

Each task must explicitly record the MCP inputs and evidence it requires. Do not leave this implicit.

- Browser-backed UI validation -> declare the required `Playwright MCP` evidence
- Runtime performance or browser security inspection -> declare the required `Chrome DevTools MCP` evidence
- External framework or library uncertainty -> declare the required `Context7` topic or package
- Figma design review -> declare `Figma MCP` only if the user actually supplied a Figma link or explicitly asked to work from Figma

If an MCP is not relevant, mark it `N/A`. If Figma was not provided, say `Not provided by user`.

### Step 6 — Assign Agents

For each task, note which agents will be involved.

### Step 7 — Order by Dependency

Sequence tasks so dependencies are respected. Mark blockers explicitly.

---

## Rules

- Tasks must not be so large that they can't be reviewed in a single QA cycle.
- Each task must have a definition of done (reference `.github/skills/definition-of-done/SKILL.md`).
- Each task must declare MCP inputs and evidence, even when the answer is `N/A`.
- Figma MCP must only be required when the user provided a Figma link or explicitly asked to use Figma.
- Never start development on a task before its upstream dependencies are complete.
- Update `tasks.md` status as work progresses — it is a living document.

---

## Usage

See `TEMPLATE.md` in this folder for the exact format to use when writing `tasks.md`.
