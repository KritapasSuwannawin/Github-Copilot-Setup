# Copilot Instructions

These instructions apply to every Copilot interaction in this project.

---

## Tech Stack

| Layer            | Technology                   |
| ---------------- | ---------------------------- |
| Frontend         | React + TypeScript           |
| State            | Redux + Redux Toolkit        |
| Data Fetching    | Native `useEffect` + `fetch` |
| Forms            | Native controlled components |
| Backend          | NestJS + TypeScript          |
| Database         | PostgreSQL + TypeORM         |
| Unit / Int Tests | Jest                         |
| E2E Tests        | Playwright                   |
| Linting          | None                         |
| Package Manager  | pnpm                         |

---

## Architecture Principles

Use **Feature-Sliced Design (FSD)** for frontend code and **Clean Architecture** for backend code. Do **not** apply **DDD** or **Hexagonal Architecture** unless the user explicitly asks for them. See `.github/skills/feature-sliced-design/SKILL.md` and `.github/skills/clean-architecture/SKILL.md`.

Frontend rule: dependencies flow from higher FSD layers to lower ones, and slices expose a small public API.
Backend rule: dependencies point inward only. `use-cases/` and `domain/` never import `controllers/` or `infrastructure/`.

### Frontend Layers (Feature-Sliced Design)

| Layer       | Responsibility                                             |
| ----------- | ---------------------------------------------------------- |
| `app/`      | App bootstrap, providers, routing, and global store wiring |
| `pages/`    | Route-level composition                                    |
| `widgets/`  | Large UI blocks composed into pages                        |
| `features/` | User actions and business interactions                     |
| `entities/` | Business entities, related state, and focused UI           |
| `shared/`   | Generic UI kit, API utilities, libraries, and config       |

### Backend Layers (Clean Architecture)

| Layer             | Responsibility                                                     |
| ----------------- | ------------------------------------------------------------------ |
| `modules/`        | Feature-specific modules                                           |
| `controllers/`    | HTTP in/out — parse requests, call use-cases, map responses        |
| `use-cases/`      | Application workflows and business orchestration                   |
| `domain/`         | Core business rules and domain types                               |
| `repositories/`   | Abstract data-access contracts required by use-cases               |
| `infrastructure/` | Framework and IO implementations such as ORM, HTTP clients, queues |
| `shared/`         | Generic utilities, libs, and config used across layers             |

---

## Coding Conventions

- **TypeScript strict mode** — `"strict": true` in tsconfig
- **No `any`** — use `unknown` and narrow, or define a proper type
- **Naming**: `camelCase` variables/functions, `PascalCase` classes/interfaces, `SCREAMING_SNAKE_CASE` constants
- **One class/component per file**; filename matches the export
- **No business logic in controllers or UI components**
- **No `console.log`** in committed code — use a logger
- **Absolute imports** — configured via tsconfig paths
- **No linting** — do not add, run, require, or rely on ESLint, TSLint, Biome, or any other linting tool in this project
- **No emojis** — always use clear text instead of emoji characters

Full standards: `.github/skills/coding-standards/SKILL.md`

---

## Agent Communication Protocol

**Agents must ask, never assume.** When faced with:

- Ambiguity in requirements or design decisions
- Missing context (e.g., no running app, no Figma link, no database state)
- Uncertainty about preferences or constraints
- Technical choices between multiple valid approaches

**Always ask the user directly.** Do not guess, infer, or proceed without clarity. Questions are fast and prevent rework.

**Core rule:** If there's ANY doubt, ask first—incomplete answers or wrong assumptions waste time and create technical debt.

---

## Task Completion Protocol

When the user asks you to implement something by referencing a file or task:

1. **Track the work** — Use the todo list tool if managing multiple steps
2. **Complete the implementation** — Follow all applicable skills, standards, and architecture principles
3. **Mark as completed immediately** — Update the todo list to show completion as soon as the task is done
4. **Confirm completion** — Briefly confirm what was finished

Do not batch completions. Mark each task as completed individually and immediately upon finishing.
