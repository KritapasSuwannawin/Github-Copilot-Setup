---
name: hexagonal-architecture
description: Optional Hexagonal Architecture reference. Not part of the default architecture in this repo; use only when the user explicitly asks for ports-and-adapters modelling.
---

# Skill: Hexagonal Architecture (Ports & Adapters)

## Purpose

This skill defines Hexagonal Architecture as an explicit opt-in reference for cases where the user asks for ports-and-adapters modelling.

This repo does **not** apply Hexagonal Architecture by default. Backend work uses Clean Architecture unless the user explicitly opts into hexagonal terminology and structure.

---

## When to Apply

Apply hexagonal architecture only when the user explicitly asks for it and:

- The system has multiple external integration points (databases, APIs, message queues, email, etc.)
- You want to test business logic without spinning up real infrastructure
- You need to swap or mock external dependencies easily

---

## Core Concept

The application has a **hexagonal core** (domain + application logic) surrounded by **ports** and **adapters**.

```
                    ┌──────────────────────┐
    HTTP Request ──▶│  Driving Adapter     │
    CLI Command ──▶ │  (Controller, CLI)   │
                    └──────────┬───────────┘
                               │ uses
                    ┌──────────▼───────────┐
                    │    Driving Port      │  ← interface the adapter calls
                    │  ┌────────────────┐  │
                    │  │  Application   │  │
                    │  │  Core (Domain) │  │
                    │  └────────────────┘  │
                    │    Driven Port       │  ← interface the core depends on
                    └──────────┬───────────┘
                               │ implemented by
                    ┌──────────▼───────────┐
                    │  Driven Adapter      │
                    │  (DB, Email, API...) │
                    └──────────────────────┘
```

---

## Ports

### Driving Ports (Primary / Left side)

- Interfaces that the application exposes to the outside world
- Implemented **by the application core**
- Examples: `ICreateOrderUseCase`, `IGetUserProfileUseCase`

### Driven Ports (Secondary / Right side)

- Interfaces the application **requires** from the outside world
- Defined in the Application layer, implemented in Infrastructure
- Examples: `IUserRepository`, `IEmailService`, `IPaymentGateway`

---

## Adapters

### Driving Adapters (Primary)

- Call into the application via driving ports
- Examples: HTTP controllers, GraphQL resolvers, CLI handlers, event consumers

### Driven Adapters (Secondary)

- Implement driven ports
- Examples: TypeORM repositories, Nodemailer email service, Stripe payment adapter

---

## Implementation Pattern (NestJS)

```typescript
// Driven Port — defined in Application layer
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

// Driven Adapter — defined in Infrastructure layer
@Injectable()
export class TypeOrmUserRepository implements IUserRepository {
  constructor(
    @InjectRepository(UserEntity)
    private readonly repo: Repository<UserEntity>,
  ) {}

  async findById(id: string): Promise<User | null> { ... }
  async save(user: User): Promise<void> { ... }
}

// Wired via NestJS DI
providers: [
  { provide: 'IUserRepository', useClass: TypeOrmUserRepository }
]
```

---

## Testing Benefit

Because all external dependencies are behind interfaces, you can test the core with in-memory adapters:

```typescript
// In-memory adapter for tests
export class InMemoryUserRepository implements IUserRepository {
  private users: Map<string, User> = new Map();

  async findById(id: string) {
    return this.users.get(id) ?? null;
  }
  async save(user: User) {
    this.users.set(user.id, user);
  }
}
```

---

## Anti-Patterns to Avoid

- Application core importing from infrastructure (breaks isolation)
- Driven ports defined in infrastructure layer (they belong in application)
- Fat adapters containing business logic
- Skipping port interfaces and calling adapters directly from use cases
