---
name: unit-testing
description: Standards and patterns for writing unit tests in TypeScript. Used by frontend-developer, backend-developer, and dev-lead during review.
---

# Skill: Unit Testing

## Purpose

Standards and patterns for writing unit tests in TypeScript — used by `frontend-developer`, `backend-developer`, and `dev-lead` during review.

---

## Principles

- **Test behaviour, not implementation** — tests should survive refactoring
- **One assertion theme per test** — each test should verify one logical thing
- **Deterministic** — tests must not depend on time, randomness, or external state
- **Independent** — tests must not rely on execution order or shared mutable state
- **Fast** — unit tests should run in milliseconds; mock all I/O

---

## What to Test

### Always test:

- Domain logic (entities, value objects, domain services)
- Use cases / application services
- Pure utility functions
- Data transformations and mappers
- Validation logic
- Error paths and edge cases

### Do not unit test:

- Framework wiring (NestJS module setup, dependency injection)
- Database queries (test these in integration tests)
- Third-party library behaviour
- Simple getters/setters with no logic

---

## Naming Convention

```
describe('{Unit under test}', () => {
  describe('{method or scenario}', () => {
    it('should {expected behaviour} when {condition}', () => { ... });
  });
});
```

Example:

```typescript
describe('CreateUserUseCase', () => {
  describe('execute', () => {
    it('should create a user when input is valid', async () => { ... });
    it('should throw UserAlreadyExistsError when email is taken', async () => { ... });
  });
});
```

---

## AAA Pattern

Every test follows Arrange → Act → Assert:

```typescript
it("should return the total price including tax", () => {
  // Arrange
  const order = new Order({ subtotal: 100, taxRate: 0.1 });

  // Act
  const total = order.calculateTotal();

  // Assert
  expect(total).toBe(110);
});
```

---

## Mocking

Mock all external dependencies at the boundary of what you're testing.

### Backend (Jest)

```typescript
const mockUserRepository: jest.Mocked<IUserRepository> = {
  findById: jest.fn(),
  findByEmail: jest.fn(),
  save: jest.fn(),
  delete: jest.fn(),
};

const useCase = new CreateUserUseCase(mockUserRepository);
```

### Frontend (Jest + Testing Library)

```typescript
// Mock a service module
jest.mock("../services/userService", () => ({
  getUser: jest.fn().mockResolvedValue({ id: "1", name: "Alice" }),
}));

// Mock a custom hook
jest.mock("../hooks/useUserProfile", () => ({
  useUserProfile: () => ({
    profile: mockProfile,
    isLoading: false,
    error: null,
  }),
}));
```

---

## Frontend Component Testing

Use **React Testing Library** — test what users see and do, not component internals.

```typescript
import { render, screen, fireEvent } from '@testing-library/react';

it('should show error message when form is submitted empty', async () => {
  render(<LoginForm />);

  fireEvent.click(screen.getByRole('button', { name: /sign in/i }));

  expect(await screen.findByText(/email is required/i)).toBeInTheDocument();
});
```

Rules:

- Query by role, label, or text — not by CSS class or test ID (except for non-semantic elements)
- Use `userEvent` from `@testing-library/user-event` for realistic interaction simulation
- Do not test internal state — test what the user sees

---

## Coverage Expectations

| Layer                            | Minimum Coverage        |
| -------------------------------- | ----------------------- |
| Domain (entities, value objects) | 90%+                    |
| Use cases / application services | 85%+                    |
| Utility functions                | 90%+                    |
| React hooks with business logic  | 80%+                    |
| UI components                    | 70%+ (key interactions) |
| Controllers / adapters           | 60%+ (error paths)      |

Coverage is a signal, not a goal — 100% coverage with poor tests is worse than 70% with thorough ones. Configure Jest with `coverageThreshold` in `jest.config.ts` to enforce minimums on CI and prevent regressions.

---

## Snapshot Testing

Snapshot tests capture component output and fail if it changes unexpectedly. Use them with care:

**When to use:**

- Stable, highly visual presentational components where the exact output matters (e.g. a date formatter, a badge variant)
- Serialising output that is complex to assert field-by-field

**When to avoid:**

- Components that change frequently during development — snapshots become noise and get blindly updated
- Any component containing business logic — test the logic explicitly, not the snapshot
- Large component trees — snapshots become unreadable and mask real regressions

**Rules:**

- Always review snapshot diffs in code review — a changed snapshot is not automatically correct
- Store snapshots in the same folder as the test file
- Never commit auto-updated snapshots without understanding the change

---

## Test File Organisation

- Co-locate test files: `user.service.ts` → `user.service.spec.ts`
- Group related tests in `describe` blocks
- Use `beforeEach` for setup, `afterEach` for teardown
- Never use `beforeAll` / `afterAll` with shared mutable state

---

## Anti-Patterns to Avoid

- Testing implementation details (private methods, internal state)
- Tests that pass when the feature is broken
- Snapshot tests for logic (use explicit assertions)
- Testing multiple unrelated behaviours in one test
- Skipping tests with `xit` / `it.skip` without a comment and issue link
