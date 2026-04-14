---
name: api-design
description: REST API design conventions for architect-lead and backend-developer. Ensures consistency across all endpoints and predictable contracts for the frontend.
---

# Skill: API Design

## Purpose

This skill defines REST API design conventions for use by `architect-lead` and `backend-developer`. Following these ensures consistency across all endpoints and a predictable contract for the frontend.

---

## URL Conventions

- Use **nouns**, not verbs: `/users`, not `/getUsers`
- Use **kebab-case** for multi-word resources: `/user-profiles`
- Use **plural** resource names: `/orders`, not `/order`
- Nest resources to show ownership (max 2 levels deep):
  - `/users/:userId/orders` [GOOD]
  - `/users/:userId/orders/:orderId/items/:itemId` [BAD] — too deep, flatten it
- Use query params for filtering, sorting, and pagination: `/orders?status=pending&sort=createdAt&order=desc`

---

## HTTP Methods

| Method   | Usage                       | Idempotent? |
| -------- | --------------------------- | ----------- |
| `GET`    | Read resource(s)            | Yes         |
| `POST`   | Create a new resource       | No          |
| `PUT`    | Replace a resource entirely | Yes         |
| `PATCH`  | Partial update              | No          |
| `DELETE` | Remove a resource           | Yes         |

---

## HTTP Status Codes

| Code                        | When to Use                                   |
| --------------------------- | --------------------------------------------- |
| `200 OK`                    | Successful GET, PATCH, PUT                    |
| `201 Created`               | Successful POST (include `Location` header)   |
| `204 No Content`            | Successful DELETE                             |
| `400 Bad Request`           | Validation error, malformed input             |
| `401 Unauthorized`          | Not authenticated                             |
| `403 Forbidden`             | Authenticated but not authorised              |
| `404 Not Found`             | Resource does not exist                       |
| `409 Conflict`              | Conflict with existing state (e.g. duplicate) |
| `422 Unprocessable Entity`  | Semantic validation error                     |
| `500 Internal Server Error` | Unexpected server error                       |

---

## Request & Response Shapes

All responses use a unified envelope so frontend consumers iterate a single shape:

```typescript
interface ApiResponse<T> {
  data: T | null;
  error: ApiError | null;
}
```

### Success Response

```json
{ "data": { ... }, "error": null }
```

### Paginated Response

```json
{
  "data": [ ... ],
  "meta": {
    "total": 100,
    "page": 1,
    "limit": 20,
    "totalPages": 5
  },
  "error": null
}
```

### Error Response

```json
{
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human-readable summary",
    "details": [
      { "field": "email", "message": "Must be a valid email address" }
    ]
  }
}
```

---

## Authentication Scheme

- All protected endpoints require a `Authorization: Bearer <jwt>` header.
- JWTs are issued on login and must be validated (signature + expiry) on every request via `JwtAuthGuard`.
- Refresh tokens are stored in `httpOnly` cookies; access tokens are short-lived (15 minutes).
- Public endpoints must be explicitly marked with `@Public()` decorator (or equivalent) — protection is the default, not an opt-in.

---

## OpenAPI Documentation

Every new endpoint must be documented with NestJS Swagger decorators:

```typescript
@ApiTags('users')
@ApiOperation({ summary: 'Create a new user' })
@ApiResponse({ status: 201, description: 'User created', type: UserResponseDto })
@ApiResponse({ status: 409, description: 'Email already exists' })
@Post()
async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> { ... }
```

Rules:

- Every endpoint must have at least one `@ApiResponse` for each documented status code.
- DTOs used as request bodies or responses must use `@ApiProperty` on each field.
- The Swagger UI must remain up to date and browsable at `/api/docs`.

---

## Versioning

- Version via URL prefix: `/api/v1/users`
- Never break an existing version — add a new one
- Deprecate old versions with a `Deprecation` response header

---

## Validation

- Validate all inputs at the controller layer using DTOs
- Never trust client data — validate types, lengths, formats, and ranges
- Return `400` with field-level error details for validation failures

---

## Error Handling

- Never expose stack traces or internal details in API responses
- Use consistent error codes (strings, not just HTTP codes): `USER_NOT_FOUND`, `INSUFFICIENT_BALANCE`
- Log full error details server-side; return minimal details to client

---

## Security Conventions

- Always require authentication on non-public endpoints
- Return `404` instead of `403` when a resource exists but the user shouldn't know about it
- Sanitise all string inputs to prevent injection
- Rate-limit sensitive endpoints (login, password reset)

---

## Anti-Patterns to Avoid

- Verbs in URLs (`/createUser`, `/deleteOrder`)
- Returning `200` for errors
- Exposing internal IDs or database row numbers as public identifiers
- Inconsistent field naming (mix of camelCase and snake_case)
- Deeply nested response objects
