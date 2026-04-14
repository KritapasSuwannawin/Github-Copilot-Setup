# AGENTS.md — Agent System Reference

> For global coding conventions and stack info, see `.github/copilot-instructions.md`.
> This file is the reference manual for the agent orchestration system.

---

## Output Review Checkpoint

**All agents pause after writing `.md` files to `.github/docs/`.** Before proceeding to the next step:

1. Agent lists all documentation files created/modified
2. Agent waits for user's explicit approval
3. User reviews files in the editor and may edit them directly
4. User replies to approve, then agent continues

See `.github/instructions/agent-output-review.instructions.md` for implementation details.

---

## Agent Roster

Agents are available at `.github/agents/`. Each agent has a specific role and responsibilities.

| Agent                | Role                                                                                     |
| -------------------- | ---------------------------------------------------------------------------------------- |
| `project-manager`    | Entry point. Coordinates all agents. Owns the task breakdown.                            |
| `architect-lead`     | Decides which architecture principles apply and produces architecture docs.              |
| `designer-ui-ux`     | Produces UI/UX designs, wireframes, and component specs.                                 |
| `dev-lead`           | Governs frontend and backend developers. Performs code review and test review.           |
| `frontend-developer` | Implements frontend features. Writes unit and E2E tests.                                 |
| `backend-developer`  | Implements backend features. Writes unit and integration tests.                          |
| `qa-lead`            | Reviews developer test evidence, governs specialist testers, and owns pass/fail verdict. |
| `tester-security`    | Audits for vulnerabilities and security issues.                                          |
| `tester-performance` | Audits frontend and backend performance.                                                 |
| `tester-usability`   | Evaluates UX quality and usability heuristics.                                           |

---

## MCP Usage Matrix

| MCP                   | Primary agents                                                            | Use when                                                                                                                             | Required evidence                                                                                                                 |
| --------------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| `Playwright MCP`      | `frontend-developer`, `tester-usability`                                  | A task needs browser-backed validation of user flows, UI states, or reproducible issue replay                                        | Screenshots for key states, traces on failure, and video for hard-to-reproduce or long-running flows when requested in `tasks.md` |
| `Chrome DevTools MCP` | `tester-performance`, `tester-security`, `frontend-developer`, `dev-lead` | A running application needs runtime inspection for rendering, networking, console, performance, or browser-visible security behavior | Performance trace, network or header evidence, console snapshot, and memory or Lighthouse/CWV snapshot when relevant              |
| `Context7`            | `architect-lead`, `dev-lead`, `frontend-developer`, `backend-developer`   | A task depends on external framework or library APIs, new dependencies, or ambiguous vendor guidance                                 | Library, version or docs ID, and topic consulted in `architecture.md`, `dev-summary.md`, or review notes                          |
| `Figma MCP`           | `designer-ui-ux`, `tester-usability`, `frontend-developer`                | The user provides a Figma link or explicitly asks to work from Figma                                                                 | Cited frame, component, or token references in `ui-spec.md`, plus comparison evidence when design parity is reviewed              |

## MCP Evidence Policy

- `project-manager` records required MCP inputs and evidence per task in `tasks.md`.
- Downstream agents must either attach the required evidence or explicitly document why the input was unavailable.
- `qa-lead` must publish an `Evidence Index` in every QA report so reviewers can trace findings back to artifacts or source references.
- Missing inputs are not a reason to guess. If the app is not running, the Figma link was not provided, or the required environment is unavailable, the agent must say so clearly.

---

## Orchestration Workflow

### Standard Workflow

```
User
 └─▶ project-manager
       ├─▶ architect-lead     → .github/docs/{feature}/architecture.md
       ├─▶ plan               → overview plan
       ├─▶ [task breakdown]   → .github/docs/{feature}/tasks.md
       └─▶ per task:
             ├─▶ designer-ui-ux (skip if task has no user-facing UI)    → .github/docs/{feature}/{task}/ui-spec.md
             ├─▶ dev-lead
             │     ├─▶ frontend-developer → implementation + unit/E2E tests
             │     ├─▶ backend-developer  → implementation + unit/integration tests
             │     └─▶ [code review + test review by dev-lead]          → .github/docs/{feature}/{task}/dev-summary.md
             └─▶ qa-lead
                   ├─▶ tester-security
                   ├─▶ tester-performance
                   ├─▶ tester-usability
                   └─▶ [consolidate QA report + verdict by qa-lead]     → .github/docs/{feature}/{task}/qa-report.md
                         └─▶ PASS → next task
                         └─▶ FAIL (cycle 1-2)   → back to dev-lead
                         └─▶ FAIL (cycle 3+)    → escalate to architect-lead
```

### Hotfix / Fast-Track Workflow

For low-risk, urgent fixes that do not introduce new features or change architecture:

```
User
 └─▶ project-manager (fast-track mode)
       ├─▶ dev-lead
       │     ├─▶ frontend-developer / backend-developer (+ required E2E/integration tests)
       │     └─▶ [code review by dev-lead]
       └─▶ qa-lead (reduced scope)
             ├─▶ tester-security    (if auth or data is touched)
             └─▶ [verdict]
```

Fast-track is permitted only when **all** of the following are true:

- No new user flows or screens
- No schema changes
- No architecture changes
- `project-manager` explicitly declares fast-track in the task brief

### QA Fail Escalation Rule

If the dev→QA loop fails twice for the same task, `project-manager` **must** invoke `architect-lead` to review the design before allowing a third dev attempt. Do not retry indefinitely without architectural review.

---

## Docs Folder Convention

All agent-produced documents are stored in `.github/docs/` following this structure:

```
.github/docs/
└── {feature}/                      ← one-line summary of the user's input to project-manager
    ├── architecture.md             ← produced by architect-lead (once per feature)
    ├── tasks.md                    ← produced by project-manager (once per feature)
    └── {task}/                     ← short slug of the task title from tasks.md
        ├── ui-spec.md              ← produced by designer-ui-ux
        ├── dev-summary.md          ← produced by dev-lead
        └── qa-report.md            ← produced by qa-lead
```

### Naming Rules

- `{feature}` — kebab-case slug of the user's original request to `project-manager`
  - Example: `"Add user authentication"` → `user-authentication`
- `{task}` — kebab-case slug of the task title from `tasks.md`
  - Example: `"Task 1.2 — Login form UI"` → `task-1-2-login-form-ui`

### Usage Rules

- The `project-manager` creates the `{feature}/` folder and writes `tasks.md` before any delegation.
- Each agent writes its output to the correct path before handing off.
- Downstream agents always read upstream docs from this folder — not from memory or inline summaries.
- `architecture.md` is written once per feature and referenced by all tasks within it.
- `qa-report.md` is updated in-place if the dev→QA loop iterates.

---

## Context Window Guidance

Large documents (`tasks.md`, `architecture.md`) can exceed an agent's context window. Follow these rules:

- Agents must reference specific **sections** of upstream docs, not include entire files inline.
- For `architecture.md`, reference only the relevant layer section (e.g. "Backend Architecture" section).
- For `tasks.md`, reference only the specific task being worked on.
- If a document is too large to pass in full, summarise the relevant parts in the delegation message.
- Paths are **always relative to the workspace root**. The `.github/` folder path must never be changed without updating every agent file and every skill file that references it.

---

## Context Handoff Protocol

When delegating, agents pass context by referencing docs paths explicitly:

```
@{agent-name}

**Task:** {task title}
**Docs:**
- Architecture: `.github/docs/{feature}/architecture.md`
- UI Spec: `.github/docs/{feature}/{task}/ui-spec.md`
- Task details: `.github/docs/{feature}/tasks.md` → Task {id}
**MCP Inputs:** {running app URL, Figma link if provided, external libraries to verify, or `none`}
**Required Evidence:** {screenshots, traces, performance profile, docs consulted, or `none`}
**Goal:** {what you need from this agent}
**Constraints:** {any decisions already made}
```

---

## Skills Library

All agents reference skills from `.github/skills/` in explicit workflow steps rather than optional top-level lists. Keep this matrix aligned with the agent files.

| Skill                   | Used By                                              |
| ----------------------- | ---------------------------------------------------- |
| `task-breakdown`        | project-manager                                      |
| `definition-of-done`    | project-manager, qa-lead                             |
| `clean-architecture`    | architect-lead, backend-developer                    |
| `feature-sliced-design` | architect-lead, frontend-developer                   |
| `api-design`            | architect-lead, backend-developer                    |
| `database-design`       | architect-lead, backend-developer                    |
| `coding-standards`      | dev-lead, frontend-developer, backend-developer      |
| `frontend-patterns`     | frontend-developer                                   |
| `backend-patterns`      | backend-developer                                    |
| `unit-testing`          | frontend-developer, backend-developer, dev-lead      |
| `integration-testing`   | backend-developer                                    |
| `e2e-testing`           | frontend-developer                                   |
| `code-review`           | dev-lead                                             |
| `security-review`       | tester-security                                      |
| `performance-review`    | tester-performance                                   |
| `usability-review`      | tester-usability                                     |
| `accessibility`         | designer-ui-ux, frontend-developer, tester-usability |
| `ui-design`             | designer-ui-ux, tester-usability                     |
| `design-system`         | designer-ui-ux, frontend-developer                   |
| `state-management`      | frontend-developer                                   |
| `migration`             | backend-developer                                    |
