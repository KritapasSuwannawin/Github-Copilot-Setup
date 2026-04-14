---
name: definition-of-done
description: The Definition of Done (DoD) is the shared standard that determines when a task is truly complete. All agents must apply these criteria. Used by qa-lead as the final checklist before issuing a verdict.
---

# Skill: Definition of Done

## Purpose

The Definition of Done (DoD) is the shared standard that determines when a task is truly complete. All agents must apply these criteria. The `qa-lead` uses this as the final checklist before issuing a verdict.

---

## Universal DoD (All Tasks)

### Code Quality

- [ ] Code follows coding standards (`.github/skills/coding-standards/SKILL.md`)
- [ ] No `any` types used without explicit justification
- [ ] No commented-out code committed
- [ ] No TODO comments without a linked issue
- [ ] No console.log / debug statements left in code

### Architecture

- [ ] Implementation aligns with `architecture.md`
- [ ] Layer boundaries respected (no leaking logic across layers)
- [ ] No circular dependencies introduced

### Documentation

- [ ] Any new public API endpoints are documented (OpenAPI/Swagger annotations)
- [ ] `README.md` updated if setup steps, commands, or architecture changed
- [ ] New components have at least a usage comment or Storybook story (frontend)

### Database

- [ ] If schema changed: migration file included, named descriptively, and is reversible (`up` + `down`)

### Testing

- [ ] Unit tests written for all non-trivial logic
- [ ] Unit tests passing
- [ ] Required integration tests written and passing
- [ ] Required E2E tests written and passing
- [ ] No tests skipped without justification

### Review

- [ ] `dev-lead` code review: PASS
- [ ] `dev-lead` required test review: PASS

## MCP Evidence

- [ ] Required MCP inputs and evidence declared in `tasks.md`
- [ ] Browser-backed tasks include the required Playwright or Chrome DevTools evidence, or explicitly document why the environment was unavailable
- [ ] Tasks that rely on external framework or library behavior document the Context7 library and topic consulted
- [ ] If the user provided a Figma link, UI specs and UI QA outputs reference the relevant frames, components, or tokens; if not, Figma is explicitly marked `Not provided by user`
- [ ] QA report includes an `Evidence Index` with links, identifiers, or source references for required MCP artifacts

---

## DoD — Frontend Tasks

- [ ] Matches `ui-spec.md` — all screens and states implemented
- [ ] All UI states handled: loading, empty, error, success
- [ ] No hardcoded strings that should be i18n or config; all user-facing strings use i18n keys if the project supports localisation
- [ ] Responsive behaviour implemented per spec
- [ ] Accessibility basics met (semantic HTML, ARIA where required, keyboard nav)
- [ ] No console errors in browser
- [ ] All interactive elements have `data-testid` attributes

---

## DoD — Backend Tasks

- [ ] All API endpoints respond with correct status codes
- [ ] Input validation in place (DTOs with class-validator)
- [ ] Errors handled and mapped to appropriate HTTP responses
- [ ] No secrets or credentials in code
- [ ] Database migrations included (if schema changed), named descriptively, and reversible

---

## DoD — QA Gate

All of the above, plus:

- [ ] Required unit tests: PASS
- [ ] Required backend integration tests: PASS
- [ ] Required frontend E2E tests: PASS
- [ ] `tester-security`: PASS (if applicable)
- [ ] `tester-performance`: PASS (if applicable)
- [ ] `tester-usability`: PASS (if applicable)
- [ ] `qa-lead` final verdict: **PASS**

---

## Severity Definitions for QA Failures

| Severity | Definition                                               | Blocks PASS?              |
| -------- | -------------------------------------------------------- | ------------------------- |
| Critical | Data loss, security breach, feature broken for all users | Always                    |
| High     | Feature broken for significant use cases                 | Always                    |
| Medium   | Noticeable issue, workaround exists                      | Usually — QA Lead decides |
| Low      | Minor friction or cosmetic issue                         | No — logged as tech debt  |
