---
name: coding-standards
description: Universal coding standards for all TypeScript code in the project, both frontend and backend. All developers and reviewers must apply these.
---

# Skill: Coding Standards

## Purpose

Universal coding standards for all TypeScript code in this project — frontend and backend. All developers and reviewers must apply these.

---

## General Principles

- **Clarity over cleverness** — code is read far more than it is written
- **Explicit over implicit** — don't rely on magic or side effects
- **Small units** — functions and classes should do one thing well
- **No surprises** — functions should do exactly what their name says
- **No linting in this project** — do not add, run, require, or rely on ESLint, TSLint, Biome, or any other linting tool

---

## TypeScript

### Types

- **Never use `any`** — use `unknown` if the type is truly unknown, then narrow it
- Prefer `interface` for object shapes that may be extended; `type` for unions, intersections, and aliases
- Use `readonly` for properties that should not be mutated
- Avoid type assertions (`as`) unless you can justify them with a comment
- Enable strict mode in `tsconfig.json` — `"strict": true`

### Naming

| Element               | Convention                      | Example                          |
| --------------------- | ------------------------------- | -------------------------------- |
| Variables & functions | `camelCase`                     | `getUserById`, `totalPrice`      |
| Classes & interfaces  | `PascalCase`                    | `UserService`, `IUserRepository` |
| Enums                 | `PascalCase`                    | `OrderStatus`                    |
| Enum values           | `SCREAMING_SNAKE_CASE`          | `OrderStatus.PENDING`            |
| Constants             | `SCREAMING_SNAKE_CASE`          | `MAX_RETRY_COUNT`                |
| Files                 | `kebab-case`                    | `user-service.ts`                |
| Interfaces            | Prefix with `I`                 | `IUserRepository`                |
| Generic types         | Single uppercase or descriptive | `T`, `TEntity`, `TResponse`      |

### Functions

- Maximum 20 lines per function — extract if longer
- Maximum 3 parameters — use an options object if more are needed
- Return types must always be explicit on public functions
- Avoid side effects in functions that return values

### Classes

- Single Responsibility Principle — one reason to change
- Prefer composition over inheritance
- Keep constructors lean — only assign properties, no logic
- Private fields should be `private readonly` where possible

---

## Comments

- **Don't comment what — comment why**
- Remove commented-out code before committing
- Use JSDoc on all public methods and classes
- Inline comments should be rare — if you need one, consider renaming instead

```typescript
// [Bad]
const d = new Date(); // current date

// [Good]
const currentDate = new Date();

// [Bad]
// Loop through users and update them
for (const user of users) { ... }

// [Good] (code is self-explanatory, no comment needed)
for (const user of users) { ... }
```

---

## Error Handling

- Never swallow errors silently
- Always use typed error classes — not raw `throw new Error('string')`
- Log errors with sufficient context (which operation, which entity ID, etc.)
- At boundaries (controllers, adapters), always catch and remap to appropriate output errors

```typescript
// [GOOD] Typed domain error
export class UserNotFoundError extends Error {
  constructor(userId: string) {
    super(`User with ID ${userId} not found`);
    this.name = "UserNotFoundError";
  }
}
```

---

## Imports

- Use absolute imports (configured via `tsconfig.json` paths) — not relative `../../..`
- Group imports: external libraries → internal modules → relative imports
- Never import barrel files circularly

---

## File Structure

- One class/component per file
- File name matches the exported class/function name
- Test files co-located with source: `user.service.ts` → `user.service.spec.ts`
- **Maximum file length: 300 lines.** Files growing beyond this are usually a Single Responsibility violation. Extract into smaller, focused modules.
- **Line endings:** LF (`\n`) on all platforms. Configure `.gitattributes` to enforce LF for `.ts`, `.tsx`, `.json`, `.md` files.
- **File encoding:** UTF-8 without BOM.

---

## Forbidden Patterns

- `console.log` in committed code (use a logger)
- Adding, running, or requiring linting tools or lint-based quality gates
- `@ts-ignore` without a comment explaining why
- `!` non-null assertion without a guard check
- Mutation of function parameters
- `var` — always use `const` or `let`
- `==` — always use `===`
