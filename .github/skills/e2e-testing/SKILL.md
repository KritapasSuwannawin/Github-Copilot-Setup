---
name: e2e-testing
description: Standards and patterns for end-to-end testing with Playwright. Used by frontend-developer.
---

# Skill: E2E Testing (Playwright)

## Purpose

Standards and patterns for end-to-end testing with Playwright. Used by `frontend-developer` when implementing user-facing flows that need browser-level coverage.

## Available Tooling

- The Playwright MCP server is available to use in this environment.
- Use Playwright MCP for browser-backed validation, artifact capture, and flow debugging before or while codifying coverage in Playwright tests.

---

## What E2E Tests Cover

E2E tests verify complete user journeys from the browser's perspective:

- Critical user flows (sign up, login, core feature workflows)
- Cross-page navigation and state persistence
- Real API integration (against a running backend)
- Visual states: loading, error, empty, success

They do **not** replace unit or integration tests. Keep the E2E suite focused on high-value flows — not every edge case.

---

## Page Object Model

Every page or major UI section has a Page Object class:

```typescript
// tests/pages/LoginPage.ts
export class LoginPage {
  constructor(private readonly page: Page) {}

  async goto() {
    await this.page.goto("/login");
  }

  async fillEmail(email: string) {
    await this.page.getByTestId("login-email-input").fill(email);
  }

  async fillPassword(password: string) {
    await this.page.getByTestId("login-password-input").fill(password);
  }

  async submit() {
    await this.page.getByTestId("login-submit-button").click();
  }

  async getErrorMessage() {
    return this.page.getByTestId("login-error-message").textContent();
  }
}
```

---

## Test Structure

```typescript
// tests/e2e/auth/login.spec.ts
import { test, expect } from "@playwright/test";
import { LoginPage } from "../pages/LoginPage";
import { DashboardPage } from "../pages/DashboardPage";

test.describe("Login", () => {
  test("should log in with valid credentials", async ({ page }) => {
    const loginPage = new LoginPage(page);
    const dashboard = new DashboardPage(page);

    await loginPage.goto();
    await loginPage.fillEmail("user@example.com");
    await loginPage.fillPassword("ValidPassword1!");
    await loginPage.submit();

    await expect(page).toHaveURL("/dashboard");
    await expect(dashboard.getWelcomeMessage()).toContainText("Welcome");
  });

  test("should show error with invalid credentials", async ({ page }) => {
    const loginPage = new LoginPage(page);

    await loginPage.goto();
    await loginPage.fillEmail("user@example.com");
    await loginPage.fillPassword("wrongpassword");
    await loginPage.submit();

    const error = await loginPage.getErrorMessage();
    expect(error).toContain("Invalid credentials");
  });
});
```

---

## Selectors

Priority order for selectors:

1. `getByRole()` — most semantically meaningful
2. `getByLabel()` — for form fields
3. `getByText()` — for buttons and links with visible text
4. `getByTestId()` — when semantic selectors aren't possible

**Never use CSS classes or DOM structure as selectors** — they break on refactoring.

Developers must add `data-testid` attributes to interactive elements:

```tsx
<button data-testid="submit-order-button" type="submit">
  Place Order
</button>
```

---

## Test Independence

- Each test must be able to run in any order
- Use `beforeEach` to set up fresh state
- Use API calls (via `request` fixture) to set up test data — not by clicking through the UI
- Clean up test data in `afterEach` or use isolated test accounts

```typescript
test.beforeEach(async ({ request }) => {
  // Seed test data via API, not UI
  await request.post("/api/test/seed", { data: { scenario: "empty-cart" } });
});
```

---

## Assertions

- Use Playwright's `expect` with web-first assertions — they auto-retry
- Prefer `toBeVisible()` over checking `.isVisible()` manually
- Use `toHaveURL()` to verify navigation
- Use `toHaveText()` / `toContainText()` for content checks

```typescript
// [GOOD] Web-first (auto-retries)
await expect(page.getByTestId("success-banner")).toBeVisible();

// [BAD] Manual (no auto-retry, flaky)
expect(await page.getByTestId("success-banner").isVisible()).toBe(true);
```

---

## Configuration

```typescript
// playwright.config.ts
export default defineConfig({
  testDir: "./tests/e2e",
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  use: {
    baseURL: process.env.BASE_URL ?? "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
    video: "retain-on-failure",
  },
});
```

## Artifact Collection

- Capture screenshots for the key states declared in `tasks.md` when the task requires UI evidence.
- Keep `trace` enabled on retry or failure so broken flows are replayable by `qa-lead` and `dev-lead`.
- Retain video for hard-to-reproduce, long-running, or high-risk flows when the task calls for it.
- In the E2E report, list each artifact by flow so downstream reviewers know what it proves.

## Visual Regression

Use visual assertions for stable UI states where layout drift would be costly:

```typescript
await expect(page.getByTestId("checkout-summary")).toHaveScreenshot(
  "checkout-summary.png",
);
```

Rules:

- Prefer screenshot assertions for deterministic views: empty states, confirmation states, key layout containers, and critical responsive breakpoints.
- Stabilise dynamic data before taking screenshots. Do not baseline timestamps, random values, or live analytics widgets.
- When a Figma link exists, treat it as a review reference for parity discussions. The automated screenshot baseline still comes from approved implementation snapshots, not directly from the Figma file.

---

## File Organisation

```
tests/
├── e2e/
│   ├── auth/
│   │   ├── login.spec.ts
│   │   └── signup.spec.ts
│   ├── {feature}/
│   │   └── {flow}.spec.ts
├── pages/
│   ├── LoginPage.ts
│   └── {Feature}Page.ts
└── fixtures/
    └── testData.ts
```

---

## Anti-Patterns to Avoid

- **Never use `page.waitForTimeout()` or any fixed-duration sleep.** This is a reliability anti-pattern that causes flaky tests on slower machines or CI environments. Always use Playwright's web-first assertions (`expect(locator).toBeVisible()`, `expect(locator).toHaveText()`) or `page.waitForSelector()` / `page.waitForResponse()` which retry automatically.
- Testing every edge case in E2E (unit and integration tests do that)
- Selector chains based on CSS structure
- Tests that depend on each other's execution order
- Hardcoded credentials in test files — use environment variables
- Screenshot assertions on unstable or highly dynamic surfaces without first controlling the data
