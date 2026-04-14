---
name: feature-sliced-design
description: Feature-Sliced Design (FSD) for frontend architecture. Use when defining or implementing React frontend layers, slices, public APIs, and import boundaries.
---

# Skill: Feature-Sliced Design (FSD)

## Purpose

This skill defines the frontend architecture for this repo. Use it with `architect-lead` and `frontend-developer` when planning or implementing React code.

---

## When to Apply

Apply FSD to all frontend work in this repo unless the user explicitly requests a different frontend architecture.

- New pages, widgets, features, entities, or shared modules
- Refactors that move frontend code into clearer ownership boundaries
- Decisions about where UI, state, API, and helper code should live

---

## Core Rules

- Organise by business ownership, not by technical type alone.
- Imports flow downward by layer.
- Each slice exposes a small public API through `index.ts`.
- Keep `shared/` generic. If code knows about a business concept, it belongs in `entities/`, `features/`, `widgets/`, or `pages/`.

---

## Layer Order

Higher layers may depend on lower layers, but never the other way around.

```text
app/ -> pages/ -> widgets/ -> features/ -> entities/ -> shared/
```

- `app/` wires the application together.
- `pages/` compose route-level screens.
- `widgets/` compose larger page sections.
- `features/` implement user actions and flows.
- `entities/` own business entities, related state, and focused UI.
- `shared/` contains generic UI, libs, config, and API primitives.

Avoid sibling-slice dependencies in the same layer. If two slices need the same code, move that code lower or into `shared/` if it is truly generic.

---

## Slice Segments

Not every slice needs every segment. Create only what the task needs.

- `ui/` for presentational components owned by the slice
- `model/` for hooks, Redux logic, selectors, validation, and state transitions
- `api/` for network calls owned by the slice
- `lib/` for small internal helpers
- `config/` for slice-local constants or settings

---

## Public API Rule

- Re-export the slice's supported surface from `index.ts`.
- Import from the slice root instead of deep-importing internal files.
- If a consumer needs an internal implementation detail, either expose it intentionally or move the code to a lower layer.

Example:

```typescript
// Good
import { SignInForm, signInThunk } from "@/features/auth";

// Avoid
import { SignInForm } from "@/features/auth/ui/SignInForm";
```

---

## Folder Structure (Example structure only, not a literal file tree)

```text
src/
├── app/
│   ├── providers/
│   ├── router/
│   └── store/
├── pages/
│   └── dashboard/
│       ├── ui/
│       └── index.ts
├── widgets/
│   └── dashboard-header/
│       ├── ui/
│       └── index.ts
├── features/
│   └── auth/
│       ├── ui/
│       ├── model/
│       ├── api/
│       └── index.ts
├── entities/
│   └── user/
│       ├── ui/
│       ├── model/
│       ├── api/
│       └── index.ts
└── shared/
    ├── api/
    ├── config/
    ├── lib/
    └── ui/
```

---

## Redux Placement

- Keep global store setup in `src/app/store/`.
- Keep reducers, selectors, and thunks in the owning slice's `model/` segment.
- Register reducers from `app/store`, but do not move slice business logic into a global `store/` dumping ground.

---

## Anti-Patterns to Avoid

- Creating a top-level frontend `domain/` layer for general business logic
- Organising the whole frontend by technical folders such as `components/`, `hooks/`, and `services/` at the root
- Deep-importing another slice's internals
- Putting business-specific code into `shared/`
- Creating extra slices or layers that do not reflect a real ownership boundary
