---
name: clean-architecture
description: Clean Architecture for architect-lead and backend-developer. Use when defining or implementing backend layers, dependency direction, use-cases, repositories, and infrastructure boundaries.
---

# Skill: Clean Architecture

## Purpose

This skill defines the backend architecture for this repo. Use it with `architect-lead` and `backend-developer` when planning or implementing backend code.

Frontend work in this repo does **not** use Clean Architecture. Frontend uses `.github/skills/feature-sliced-design/SKILL.md`.

---

## When to Apply

Apply Clean Architecture to backend work that introduces or changes:

- Controllers or transport handlers
- Use-case and application workflow logic
- Domain rules
- Repository or service interfaces
- Infrastructure implementations such as ORM, HTTP clients, queues, or email

---

## The Core Rule: Dependency Rule

> Source code dependencies must point **inward only** — toward higher-level business policies.

In this repo's backend structure:

- `controllers/` depend on `use-cases/`
- `use-cases/` depend on `domain/` and `repositories/`
- `infrastructure/` implements contracts used by `use-cases/` or `repositories/`
- `domain/` does not depend on outer layers

```text
controllers/ ─────▶ use-cases/ ─────▶ domain/
                        │
                        └──────▶ repositories/ ◀────── infrastructure/
```

---

## Layers

### `domain/`

- Core business rules and domain types
- No framework dependencies
- No imports from `controllers/` or `infrastructure/`

### `use-cases/`

- One use case per backend action or workflow
- Orchestrates domain logic
- Depends on `domain/` and repository or service interfaces only

### `repositories/`

- Abstract contracts required by `use-cases/`
- Describe data access in business terms
- Contain no ORM or transport details

### `controllers/`

- HTTP or transport adaptation layer
- Parse input, call a use case, map output or errors back to the transport
- Must not contain business logic

### `infrastructure/`

- ORM implementations, HTTP clients, queues, email, and framework wiring
- Implements interfaces required by the inner layers
- Can depend on NestJS, TypeORM, and other external libraries

---

## Dependency Injection

- Outer layers inject dependencies into inner layers via constructors
- Inner layers define required interfaces; outer layers provide implementations
- Use DI containers (NestJS IoC) to wire dependencies

---

## Backend Application

```text
src/
├── controllers/      ← HTTP entry points, DTOs, response mapping
├── use-cases/        ← application workflows
├── domain/           ← business rules and domain types
├── repositories/     ← abstract contracts used by use-cases
└── infrastructure/   ← ORM, HTTP clients, queues, framework adapters
└── shared/           ← utilities, libs, and config used across layers
```

---

## Anti-Patterns to Avoid

- Importing ORM entities into use cases
- Business logic in controllers
- Business logic in infrastructure implementations
- Use cases importing from infrastructure
- Introducing DDD or Hexagonal Architecture terminology by default on backend work in this repo
