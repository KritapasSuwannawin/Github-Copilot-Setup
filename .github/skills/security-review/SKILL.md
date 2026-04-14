---
name: security-review
description: Structured security checklist for tester-security, based on OWASP Top 10 and common TypeScript/NestJS/React vulnerabilities.
---

# Skill: Security Review

## Purpose

A structured security checklist for `tester-security`, based on OWASP Top 10 and common TypeScript/NestJS/React vulnerabilities.

---

## OWASP Top 10 Checklist

### A01 — Broken Access Control

- [ ] All endpoints require authentication unless explicitly public
- [ ] Authorisation checked at the use-case level, not just the controller
- [ ] No IDOR — resource ownership verified before access (e.g. user can only access their own orders)
- [ ] Admin-only endpoints are protected at role level
- [ ] `403` returned for authorisation failures; `404` used when existence should be hidden

### A02 — Cryptographic Failures

- [ ] No sensitive data (passwords, tokens, PII) stored in plaintext
- [ ] Passwords hashed with bcrypt or argon2 — not MD5 or SHA1
- [ ] Sensitive data not logged
- [ ] HTTPS enforced in all environments
- [ ] JWT secrets are sufficiently long and random (not hardcoded)

### A03 — Injection

- [ ] All database queries use parameterised queries or ORM — no raw string concatenation
- [ ] All user input sanitised before use in queries
- [ ] No command injection via `exec`, `spawn`, or `eval` with user input
- [ ] GraphQL queries protected against injection if applicable
- [ ] All request bodies validated with a schema library (`class-validator` + `ValidationPipe`) before any business logic runs — raw unvalidated input must not reach use cases or domain objects

### A04 — Insecure Design

- [ ] Rate limiting on sensitive endpoints (login, password reset, OTP)
- [ ] Brute force protections in place for authentication flows
- [ ] Business logic cannot be manipulated by changing request parameters (e.g. negative prices)
- [ ] CSRF protection in place for all state-mutating endpoints (POST, PUT, PATCH, DELETE) when cookie-based auth is used; verify `SameSite=Strict` or `SameSite=Lax` cookie attribute is set, or a CSRF token header mechanism is implemented

### A05 — Security Misconfiguration

- [ ] No default credentials or debug flags in any environment
- [ ] HTTP security headers set: `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Strict-Transport-Security`
- [ ] CORS configured correctly — not `origin: '*'` in production
- [ ] Error responses do not expose stack traces or internal details

### A06 — Vulnerable Components

- [ ] No known CVE vulnerabilities in direct dependencies (`pnpm audit`)
- [ ] Dependencies are reasonably up to date

### A07 — Authentication Failures

- [ ] JWTs validated on every request (signature, expiry, audience)
- [ ] Refresh token rotation implemented if refresh tokens are used
- [ ] Session invalidation on logout
- [ ] Passwords have minimum complexity requirements

### A08 — Software and Data Integrity

- [ ] No user-controlled data used in `eval()`, `new Function()`, or `dangerouslySetInnerHTML`
- [ ] File uploads validated (type, size, content)

### A09 — Logging & Monitoring

- [ ] Auth failures and suspicious actions are logged
- [ ] Logs do not contain passwords, tokens, or PII
- [ ] Errors logged with sufficient context for investigation

### A10 — Server-Side Request Forgery (SSRF)

- [ ] If the backend makes HTTP requests to user-supplied URLs — those URLs are validated against an allowlist

---

## React-Specific Checks

## Chrome DevTools MCP Runtime Checks

When a running environment is available, verify browser-visible security behavior with runtime evidence:

- [ ] Capture representative authenticated and unauthenticated requests and inspect response headers
- [ ] Confirm `Content-Security-Policy`, `Strict-Transport-Security`, `X-Frame-Options`, and `X-Content-Type-Options` are present where expected
- [ ] Inspect CORS preflight behavior and verify disallowed origins or methods are rejected correctly
- [ ] Check for mixed-content requests, blocked resources, and suspicious console security warnings
- [ ] Verify error responses shown in the browser do not expose stack traces or internal details
- [ ] Record unexpected client-side network calls, third-party endpoints, or leaked secrets in requests

---

## Finding Format

### Runtime Evidence

```
## Security Review: {Task Title}

### Findings
| # | Severity | Location | Issue | Recommendation |
|---|---|---|---|---|
| 1 | Critical | `auth.controller.ts:45` | JWT not validated on endpoint | Add `@UseGuards(JwtAuthGuard)` |
| 2 | High | `user.service.ts:23` | Raw SQL with user input | Use TypeORM query builder |
| 3 | Medium | `main.ts:12` | CORS allows all origins | Restrict to known domains |

### Verdict: PASS / FAIL
```
