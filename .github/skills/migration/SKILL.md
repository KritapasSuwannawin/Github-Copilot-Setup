---
name: migration
description: Database migration standards for backend-developer using TypeORM. Covers creating, running, and reverting migrations safely in all environments.
---

# Skill: Database Migrations

## Purpose

This skill defines how to create, apply, and revert TypeORM migrations in this project. Every schema change must go through a migration — never modify the database schema by hand in staging or production.

---

## Core Rules

- **Never** use `synchronize: true` in production or staging TypeORM config
- Every schema change (column add/remove, index, constraint, type change) requires a migration file
- Migrations must be reviewed by `dev-lead` before merging
- Migrations must be reversible — always implement `down()` unless the operation is truly irreversible (document why if so)
- Test migrations locally by running `up` and then `down` before committing

---

## Generating a Migration

```bash
# After changing an entity file:
pnpm --filter backend migration:generate -- -n <MigrationName>
```

Use PascalCase for migration names that reflect the change:

```
AddUserEmailIndex
CreateOrdersTable
RenameProductSkuColumn
```

Generated files land in `src/infrastructure/database/migrations/`.

---

## Migration File Structure

```typescript
import { MigrationInterface, QueryRunner } from "typeorm";

export class AddUserEmailIndex1700000000000 implements MigrationInterface {
  name = "AddUserEmailIndex1700000000000";

  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(
      `CREATE INDEX "IDX_user_email" ON "user" ("email")`,
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX "IDX_user_email"`);
  }
}
```

---

## Running Migrations

```bash
# Apply all pending migrations
pnpm --filter backend migration:run

# Revert the last applied migration
pnpm --filter backend migration:revert

# Show migration status
pnpm --filter backend migration:show
```

---

## CI/CD Pipeline

Migrations run automatically as part of the deployment pipeline **before** the new application version starts. The pipeline must:

1. Run `migration:run`
2. Only proceed to deploy the app if migrations succeed
3. On failure, halt deployment and alert the team — do not auto-revert

---

## Destructive Operations

Destructive operations require extra care:

| Operation          | Risk                                | Safe Approach                                     |
| ------------------ | ----------------------------------- | ------------------------------------------------- |
| Drop column        | Data loss if rolled back            | Add column as nullable first, backfill, then drop |
| Rename column      | Breaks existing queries in old code | Use a two-phase deploy: add new + keep old        |
| Change column type | Data truncation risk                | Create new column, migrate data, drop old         |
| Drop table         | Irreversible data loss              | Soft-delete or archive rows before dropping       |

---

## Seeding

Seeds are separate from migrations and are only run in development and CI:

```bash
pnpm --filter backend seed
```

Seeds must be idempotent — running them twice must not create duplicate data.

---

## Testing Migrations

In integration tests, use a test database. Apply all migrations at the start of the test suite using `DataSource.runMigrations()`. Do not write integration tests against a schema created via `synchronize: true`.

```typescript
beforeAll(async () => {
  await dataSource.initialize();
  await dataSource.runMigrations();
});
```
