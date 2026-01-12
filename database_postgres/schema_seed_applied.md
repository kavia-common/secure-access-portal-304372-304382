# PostgreSQL Auth/RBAC Schema + Seed (applied via CLI)

This project requires an authentication + RBAC schema with audit timestamps.  
Per instructions, schema and seed data were applied **via `psql -c` one statement at a time** (no .sql files created).

## Connection used
From `database_postgres/db_connection.txt`:
- `psql postgresql://appuser:dbuser123@localhost:5000/myapp`

## DDL applied (one statement per call)
- `CREATE EXTENSION IF NOT EXISTS pgcrypto;`

Tables:
- `users`
  - `id uuid PK default gen_random_uuid()`
  - `email text unique not null`
  - `password_hash text not null`
  - `display_name text not null`
  - `is_active boolean not null default true`
  - `created_at timestamptz not null default now()`
  - `updated_at timestamptz not null default now()`
  - `last_login_at timestamptz null`
- `roles`
  - `id uuid PK default gen_random_uuid()`
  - `name text unique not null`
  - `description text null`
  - `created_at timestamptz not null default now()`
  - `updated_at timestamptz not null default now()`
- `user_roles`
  - `user_id uuid FK -> users(id) on delete cascade`
  - `role_id uuid FK -> roles(id) on delete cascade`
  - `created_at timestamptz not null default now()`
  - `PRIMARY KEY (user_id, role_id)`
- `refresh_tokens` (sessions)
  - `id uuid PK default gen_random_uuid()`
  - `user_id uuid FK -> users(id) on delete cascade`
  - `token_hash text unique not null`
  - `expires_at timestamptz not null`
  - `revoked_at timestamptz null`
  - `created_at timestamptz not null default now()`
  - `updated_at timestamptz not null default now()`

Indexes:
- `idx_users_email` on `users(email)`
- `idx_user_roles_user_id` on `user_roles(user_id)`
- `idx_user_roles_role_id` on `user_roles(role_id)`
- `idx_refresh_tokens_user_id` on `refresh_tokens(user_id)`
- `idx_refresh_tokens_expires_at` on `refresh_tokens(expires_at)`

## Seed data applied (one statement per call)
Roles:
- `admin`
- `user`

Users:
- `admin@example.com` (display_name: `Admin User`)
- `user@example.com` (display_name: `Normal User`)

Role assignments:
- `admin@example.com` -> `admin`
- `user@example.com` -> `user`

Note: `password_hash` values seeded are placeholder bcrypt hashes; the backend should create/update users with its own password hashing logic.

## Verification query result
A verification query confirmed:
- `admin@example.com` has `{admin}`
- `user@example.com` has `{user}`
