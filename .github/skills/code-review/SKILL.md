---
name: code-review
description: Structured checklist and process for dev-lead when reviewing code produced by frontend-developer and backend-developer.
---

# Skill: Code Review

## Purpose

A structured checklist and process for `dev-lead` when reviewing code produced by `frontend-developer` and `backend-developer`.

---

## Review Philosophy

- Review the code, not the person
- The goal is quality and knowledge sharing — not gatekeeping
- Every finding must include a suggestion, not just a criticism
- Distinguish blockers from suggestions — be explicit

---

## Severity Labels

| Label          | Meaning                                          |
| -------------- | ------------------------------------------------ |
| `[BLOCKING]`   | Must be fixed before merge                       |
| `[SUGGESTION]` | Worth considering, not required                  |
| `[QUESTION]`   | Seeking clarification, not necessarily a problem |
| `[NITPICK]`    | Minor style or preference — totally optional     |

---

## Review Checklist

### Architecture & Design

- [ ] Implementation follows `architecture.md` — layer boundaries respected
- [ ] No business logic in controllers or UI components
- [ ] Dependencies point in the correct direction (inner layers don't depend on outer)
- [ ] Interfaces used correctly — no concrete infrastructure leaking into domain/application
- [ ] New patterns are consistent with existing patterns in the codebase

### Code Quality

- [ ] Functions and classes have a single responsibility
- [ ] Naming is clear and intention-revealing
- [ ] No magic numbers or unexplained constants
- [ ] No dead code or commented-out code
- [ ] Code is DRY — but not over-abstracted
- [ ] Complex logic has an explanatory comment (not what, but why)
- [ ] Review feedback does not require adding, running, or fixing linting tools because this project does not use linting

### TypeScript

- [ ] No `any` without justification
- [ ] Return types explicit on public methods
- [ ] Proper use of generics where beneficial
- [ ] No unsafe type assertions without a guard

### Error Handling

- [ ] All async operations have error handling
- [ ] Errors are not swallowed silently
- [ ] Error messages are helpful and appropriately detailed
- [ ] Errors don't expose sensitive internal information

### Security

- [ ] No secrets or credentials in code
- [ ] User input is validated and sanitised
- [ ] No obvious injection vectors
- [ ] Authorisation checked where required

### Testing

- [ ] New code has corresponding unit tests
- [ ] Tests cover both the happy path and error cases
- [ ] Tests are readable and maintain the codebase's test patterns
- [ ] No tests skipped without justification

### Performance

- [ ] No obvious N+1 queries
- [ ] No unnecessary re-renders in React components
- [ ] Large data sets are paginated, not loaded in full

### MCP-Assisted Review

- [ ] Use workspace inspection to compare against existing project-local patterns before calling something "new"
- [ ] Use Context7 when the change depends on external library or framework APIs, new dependencies, or vendor-specific behavior
- [ ] Review Chrome DevTools evidence for browser-facing performance or runtime security claims instead of relying on prose alone
- [ ] If a Figma link was part of the task, review design evidence against that source; if no Figma link was provided, do not treat Figma as a missing requirement

---

## Review Output Format

```
## Code Review: {Task Title}

### Summary
(Overall impression — 2-3 sentences)

### Findings

**[BLOCKING]** `path/to/file.ts` line 42
Business logic found in controller — `calculateDiscount()` should be in a use case.
Suggestion: Extract to `ApplyDiscountUseCase` and inject it here.

**[SUGGESTION]** `path/to/component.tsx` line 78
This derived value is being stored in state — it could be a computed variable instead.

**[NITPICK]** `path/to/utils.ts` line 12
Naming: `d` could be more descriptive — perhaps `discountRate`?

### Verdict: APPROVED / CHANGES REQUIRED
```

---

## Anti-Patterns to Avoid in Reviews

- Vague feedback ("this is wrong", "improve this") — always be specific
- Blocking on style preferences that aren't in the coding standards
- Reviewing only for problems — call out good patterns too
- Rewriting the developer's code for them — suggest, don't replace
