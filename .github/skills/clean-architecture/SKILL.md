---
name: clean-architecture
description: Clean Architecture principles for architect-lead, dev-lead, frontend-developer, and backend-developer.
---

# Skill: Clean Architecture

## Purpose

This skill defines Clean Architecture principles for use by `architect-lead`, `dev-lead`, `frontend-developer`, and `backend-developer`.

---

## When to Apply

Apply Clean Architecture when:

- The system needs to be testable independently of frameworks, databases, or UI
- You anticipate swapping infrastructure (e.g. changing ORM, switching databases)
- You want business logic isolated from delivery mechanisms

---

## The Core Rule: Dependency Rule

> Source code dependencies must point **inward only** — toward higher-level policies.

Nothing in an inner circle can know about something in an outer circle.

```
          ┌─────────────────────────┐
          │   Frameworks & Drivers  │  ← outermost: web, DB, UI
          │  ┌───────────────────┐  │
          │  │   Interface       │  │  ← controllers, presenters, gateways
          │  │  ┌─────────────┐  │  │
          │  │  │ Application │  │  │  ← use cases
          │  │  │  ┌───────┐  │  │  │
          │  │  │  │Domain │  │  │  │  ← innermost: entities, business rules
          │  │  │  └───────┘  │  │  │
          │  │  └─────────────┘  │  │
          │  └───────────────────┘  │
          └─────────────────────────┘
```

---

## Layers

### Domain (Innermost)

- Entities and core business rules
- No framework dependencies
- No imports from outer layers
- Pure TypeScript classes

### Application

- Use cases — one per user action
- Orchestrates domain objects
- Defines interfaces (ports) for things it needs from outside (repositories, services)
- No framework dependencies

### Interface

- Controllers, resolvers, CLI handlers
- Adapts external requests to use case inputs
- Adapts use case outputs to external response formats
- Depends on Application layer only

### Infrastructure (Outermost)

- Database implementations, ORM, HTTP clients, email, queues
- Implements interfaces defined in Application layer
- Framework-specific code lives here (NestJS modules, TypeORM entities)

---

## Dependency Injection

- Outer layers inject dependencies into inner layers via constructors
- Inner layers define interfaces; outer layers provide implementations
- Use DI containers (NestJS IoC) to wire dependencies

---

## Frontend Application

```
src/
├── domain/           ← business rules, types, validation
├── application/      ← use cases as hooks or service functions
├── infrastructure/   ← API clients, storage adapters
└── ui/               ← React components (outermost)
```

## Backend Application

```
src/
├── domain/           ← entities, value objects, domain services
├── application/      ← use cases, port interfaces
├── infrastructure/   ← ORM, HTTP clients, adapters
└── interface/        ← controllers, DTOs
```

---

## Anti-Patterns to Avoid

- Importing ORM entities into use cases
- Business logic in controllers
- Use cases importing from infrastructure
- Framework decorators on domain entities
