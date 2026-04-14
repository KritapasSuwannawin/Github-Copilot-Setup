---
name: database-design
description: Database design principles for architect-lead and backend-developer. Covers schema design, naming conventions, migrations, indexing, and common patterns.
---

# Skill: Database Design

## Purpose

This skill defines database design principles for use by `architect-lead` and `backend-developer`. It covers schema design, naming conventions, migrations, indexing, and common patterns.

---

## Naming Conventions

| Element         | Convention                | Example                    |
| --------------- | ------------------------- | -------------------------- |
| Tables          | `snake_case`, plural      | `user_profiles`            |
| Columns         | `snake_case`              | `created_at`, `first_name` |
| Primary keys    | `id` (UUID preferred)     | `id uuid`                  |
| Foreign keys    | `{table_singular}_id`     | `user_id`, `order_id`      |
| Junction tables | `{table_a}_{table_b}`     | `user_roles`               |
| Indexes         | `idx_{table}_{column(s)}` | `idx_orders_user_id`       |

---

## Primary Keys

- Prefer **UUIDs** over auto-increment integers for public-facing IDs
  - Prevents enumeration attacks
  - Safe for distributed systems
- Use `uuid_generate_v4()` in PostgreSQL or equivalent

---

## Standard Columns (Every Table)

```sql
id         UUID PRIMARY KEY DEFAULT uuid_generate_v4()
created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
```

- Add `deleted_at TIMESTAMPTZ` for soft-delete patterns

---

## Normalisation Guidelines

- Aim for **3rd Normal Form (3NF)** as a baseline
- Denormalise deliberately and document the reason when done for performance
- Avoid storing derived data that can be computed — unless it's a significant performance win

---

## Relationships

### One-to-Many

- Foreign key on the "many" side
- Always index the foreign key column

### Many-to-Many

- Use a junction table
- The junction table can have its own columns (e.g. `role`, `assigned_at`)

### One-to-One

- Foreign key with a `UNIQUE` constraint on the referencing table

---

## Indexing

### Always index:

- Foreign keys
- Columns used in `WHERE` clauses frequently
- Columns used in `ORDER BY` with large datasets
- Columns used in `JOIN` conditions

### Consider composite indexes for:

- Queries filtering on multiple columns together
- Covering indexes (include all queried columns)

### Avoid over-indexing:

- Every index slows down `INSERT`/`UPDATE`/`DELETE`
- Index columns with high cardinality (many distinct values)
- Don't index boolean columns with low cardinality

---

## Migrations

- Every schema change must be a migration file — never edit the DB manually
- Migrations must be **reversible** (include `up` and `down`)
- Name migrations descriptively: `20240101_add_status_to_orders.ts`
- Never modify an existing migration after it has been applied to any environment
- Test migrations on a copy of production data before deploying

---

## Soft Deletes

Use soft deletes when:

- Data has audit/compliance requirements
- Related records reference the deleted item

Implementation:

```sql
deleted_at TIMESTAMPTZ NULL DEFAULT NULL
```

- Filter all queries with `WHERE deleted_at IS NULL`
- Use a global query scope/filter to avoid forgetting

---

## Common Patterns

### Audit Log Table

```sql
CREATE TABLE audit_logs (
  id         UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  table_name TEXT NOT NULL,
  record_id  UUID NOT NULL,
  action     TEXT NOT NULL,  -- INSERT, UPDATE, DELETE
  changed_by UUID REFERENCES users(id),
  changed_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  old_values JSONB,
  new_values JSONB
);
```

### Status Enum Columns

- Use a `TEXT` column with a `CHECK` constraint, or a proper `ENUM` type
- Document all valid values in a comment on the column

---

## Row-Level Security (RLS)

PostgreSQL Row-Level Security can enforce tenant or user isolation at the database level, providing a defense-in-depth layer beyond application-level checks.

### When to Consider RLS

- Multi-tenant databases where rows are scoped to an `organization_id` or `user_id`
- Any table containing PII that must never cross tenant boundaries

### Minimal Pattern

```sql
-- Enable RLS on the table
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Allow users to see only their own rows
CREATE POLICY orders_tenant_isolation
  ON orders
  USING (organization_id = current_setting('app.current_org_id')::uuid);
```

The application sets `app.current_org_id` at the start of each database transaction via TypeORM query runner or a middleware.

### Notes

- RLS is complementary to, not a replacement for, application-level authorization
- Bypassed by superuser connections — service accounts must not be superusers
- Test RLS policies explicitly in integration tests by asserting cross-tenant reads return empty

---

## Anti-Patterns to Avoid

- Storing JSON blobs for data that needs to be queried
- EAV (Entity-Attribute-Value) pattern without strong justification
- Using `TEXT` for everything instead of appropriate types
- Missing foreign key constraints
- Not indexing foreign keys
- Storing passwords in plaintext (hash with bcrypt/argon2)
- Storing sensitive data unencrypted
