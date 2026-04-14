---
name: ddd
description: Optional Domain-Driven Design reference. Not part of the default architecture in this repo; use only when the user explicitly asks for DDD in a complex backend domain.
---

# Skill: Domain-Driven Design (DDD)

## Purpose

This skill provides the principles and patterns of Domain-Driven Design as an explicit opt-in reference.

This repo does **not** apply DDD by default. Use this skill only when the user explicitly requests DDD or when extending backend code that already uses DDD concepts heavily.

---

## When to Apply DDD

Apply DDD only when the user explicitly asks for it and:

- The domain has complex business rules that can't be trivially modelled as CRUD
- There are multiple distinct problem areas (bounded contexts)
- Domain experts exist and their language should be reflected in the code
- The system is expected to grow significantly in complexity

**Do not apply** DDD to standard frontend work, routine backend features in this repo, simple CRUD features, configuration screens, or pure infrastructure concerns.

---

## Core Concepts

### Ubiquitous Language

- The same terms must be used in code, documentation, and conversation
- Defined per bounded context — the same word can mean different things in different contexts
- Capture it in a glossary

### Bounded Context

- A logical boundary within which a domain model is consistent
- Each bounded context has its own model, its own ubiquitous language
- Contexts communicate via well-defined interfaces (events, APIs, anti-corruption layers)

### Entities

- Objects with a unique identity that persists over time
- Identity matters more than attribute values
- Example: `User`, `Order`, `Product`

### Value Objects

- Immutable objects defined entirely by their attributes — no identity
- Replace with a new instance when changed
- Example: `Money`, `Address`, `EmailAddress`

### Aggregates

- A cluster of entities and value objects with a single root entity
- All external access goes through the Aggregate Root
- Enforces invariants and consistency within its boundary
- Keep aggregates small — one to a few entities max

### Domain Services

- Logic that doesn't naturally belong to an entity or value object
- Stateless operations that coordinate multiple domain objects
- Example: `TransferService`, `PricingService`

### Repositories

- Provide collection-like access to aggregates
- Abstract storage — the domain doesn't know about databases
- One repository per aggregate root

### Domain Events

- Something meaningful that happened in the domain
- Named in past tense: `OrderPlaced`, `UserRegistered`
- Used to communicate between bounded contexts or trigger side effects

### Application Services (Use Cases)

- Orchestrate domain objects to fulfil a user intent
- No business logic here — delegate to domain
- Handle transactions and cross-cutting concerns

---

## Folder Structure (DDD + NestJS)

```
src/
├── {bounded-context}/
│   ├── domain/
│   │   ├── entities/
│   │   ├── value-objects/
│   │   ├── aggregates/
│   │   ├── events/
│   │   ├── services/
│   │   └── repositories/        ← interfaces only
│   ├── application/
│   │   └── use-cases/
│   ├── infrastructure/
│   │   ├── persistence/         ← repository implementations
│   │   └── adapters/            ← external services
│   └── interface/
│       └── controllers/
```

---

## Anti-Patterns to Avoid

- Anemic domain model (entities with only getters/setters, all logic in services)
- Fat controllers with business logic
- Bypassing the aggregate root to modify child entities
- Sharing a single domain model across bounded contexts
