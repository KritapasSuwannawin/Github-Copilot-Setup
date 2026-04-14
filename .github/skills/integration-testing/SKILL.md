---
name: integration-testing
description: Standards for integration tests. Verifies that components work correctly together at their boundaries. Used by tester-integration.
---

# Skill: Integration Testing

## Purpose

Standards for integration tests — verifying that components work correctly together at their boundaries. Used by `tester-integration`.

---

## What Integration Tests Cover

Integration tests verify:

- Controller → Use Case → Repository → Database flow (end-to-end within the backend)
- API request/response contracts (does the endpoint behave as documented?)
- Service-to-service communication boundaries
- Database query correctness (do queries return the right data?)
- Transaction handling (do multi-step operations roll back on failure?)

They do **not** test:

- Unit-level business logic (that's for unit tests)
- Full UI flows (that's for E2E tests)
- External third-party services (mock those at the boundary)

---

## Backend Integration Tests (NestJS)

Use NestJS testing utilities to spin up a partial application context:

```typescript
describe("UserController (integration)", () => {
  let app: INestApplication;
  let dataSource: DataSource;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule], // or a test-specific module
    }).compile();

    app = module.createNestApplication();
    app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
    await app.init();

    dataSource = module.get(DataSource);
  });

  afterAll(async () => {
    await dataSource.destroy();
    await app.close();
  });

  afterEach(async () => {
    // Clean up test data between tests
    await dataSource.query("DELETE FROM users WHERE email LIKE 'test+%'");
  });

  it("POST /users — should create a user and return 201", async () => {
    const response = await request(app.getHttpServer())
      .post("/users")
      .send({ email: "test+alice@example.com", name: "Alice" })
      .expect(201);

    expect(response.body.data).toMatchObject({
      email: "test+alice@example.com",
      name: "Alice",
    });
    expect(response.body.data.id).toBeDefined();
  });

  it("POST /users — should return 409 when email already exists", async () => {
    await createTestUser({ email: "test+bob@example.com" });

    await request(app.getHttpServer())
      .post("/users")
      .send({ email: "test+bob@example.com", name: "Bob" })
      .expect(409);
  });
});
```

---

## Test Database Strategy

- Use a **dedicated test database** — never run integration tests against staging or production
- Reset state between tests — either truncate tables or use transactions that roll back
- Use realistic seed data — not just IDs and empty strings
- Run migrations before the test suite starts

---

## API Contract Testing

For every endpoint, test:

- **Happy path** — correct input, expected response shape and status
- **Validation errors** — missing/invalid fields → `400` with field details
- **Auth** — unauthenticated requests → `401`; unauthorised → `403`
- **Not found** — non-existent resource → `404`
- **Conflict** — duplicate/constraint violation → `409`

---

## Frontend–Backend Contract

When frontend and backend are developed in parallel:

- Define the API contract in an OpenAPI spec or shared TypeScript types first
- Validate both sides against the agreed contract in integration tests
- If using React Query or SWR on the frontend, write tests that mock the API and verify the hook returns the right shape

---

## Anti-Patterns to Avoid

- Integration tests that call real external APIs (mock them)
- Tests that leave data in the database after running
- Testing the same logic covered by unit tests (don't duplicate)
- Hardcoded database connection strings (use environment config)
