---
name: surrealkit
description: "Use SurrealKit, SurrealDB's schema-management and migration CLI, to scaffold projects from templates, sync schema in development, plan and execute production rollouts (with rollback), generate JSON/TypeScript types from a live database, and write declarative TOML test suites for schemas, permissions, and API endpoints. Use this skill whenever users set up, migrate, type, or test a SurrealDB schema with SurrealKit."
metadata:
  author: surrealdb
  version: "0.1.1"
---

# SurrealKit

A skill for driving [SurrealKit](https://github.com/surrealdb/surrealkit), SurrealDB's schema-management and migration CLI.

SurrealKit keeps a SurrealDB database in sync with `.surql` schema files. It provides two complementary workflows — a fast declarative **sync** for development and controlled, phased **rollouts** for shared and production databases — plus seeding, type generation, and a declarative testing framework.

## When to use this skill

Reference these guidelines when:

- Scaffolding a new SurrealDB project (`surrealkit init`) or authoring templates
- Applying schema changes in development (`surrealkit sync`)
- Planning, executing, or rolling back production migrations (`surrealkit rollout`)
- Generating JSON or TypeScript types from a live schema (`surrealkit typegen`)
- Writing or running declarative tests for schemas, permissions, or API endpoints (`surrealkit test`)

This skill covers the SurrealKit tool itself. To write the actual schema, seed, and query statements that go in `.surql` files, use the **surrealql** skill.

## Installation

| Method | Command |
| --- | --- |
| `cargo binstall` (recommended) | `cargo binstall surrealkit` |
| Cargo (from source) | `cargo install surrealkit` |
| Docker | `docker pull ghcr.io/surrealdb/surrealkit:latest` |
| Prebuilt tarball | [GitHub Releases](https://github.com/surrealdb/surrealkit/releases) |

## Command map

| Command | Purpose | Reference |
| --- | --- | --- |
| `surrealkit init` | Scaffold a project from a template, selecting optional features | [init-templates.md](references/init-templates.md) |
| `surrealkit sync` | Declaratively reconcile the database to your schema files (dev) | [sync-rollouts.md](references/sync-rollouts.md) |
| `surrealkit rollout <sub>` | Plan, stage, complete, and roll back migrations (shared/prod) | [sync-rollouts.md](references/sync-rollouts.md) |
| `surrealkit typegen` | Introspect a live DB and emit JSON / TypeScript types | [typegen.md](references/typegen.md) |
| `surrealkit test` | Run declarative TOML test suites | [testing.md](references/testing.md) |
| `surrealkit seed` | Run seeding files in `database/seed/` | [sync-rollouts.md](references/sync-rollouts.md) |
| `surrealkit apply <path>` | Apply a single `.surql` file directly | — |
| `surrealkit status` | Show sync/rollout state | — |

Run `surrealkit <command> --help` to confirm available flags for an installed version.

## Connection & config

Global flags work on every command and resolve in this order (highest wins):
**CLI flags > system env vars > `.env` file > defaults**.

```bash
surrealkit --host http://localhost:8000 --ns my_ns --db my_db \
  --user root --pass root --auth-level root sync
```

| Flag | Env var (with fallback) | Default |
| --- | --- | --- |
| `--host` | `SURREALDB_HOST` (`DATABASE_HOST`) | `http://localhost:8000` |
| `--ns` | `SURREALDB_NAMESPACE` (`DATABASE_NAMESPACE`) | `db` |
| `--db` | `SURREALDB_NAME` (`DATABASE_NAME`) | `test` |
| `--user` | `SURREALDB_USER` (`DATABASE_USER`) | `root` |
| `--pass` | `SURREALDB_PASSWORD` (`DATABASE_PASSWORD`) | `root` |
| `--auth-level` | `SURREALDB_AUTH_LEVEL` (`DATABASE_AUTH_LEVEL`) | `root` (`root` / `namespace` / `database`) |
| `--folder` | `SURREALDB_FOLDER` | `./database` |

The project root holds `surrealkit.toml` with `[variables]` and `[typegen]` sections.

## Template variables

Use `${VAR_NAME}` tokens in any `.surql` file (schema, seed, or rollout SQL). Names are case-insensitive. Values resolve in order (highest wins):

1. `--var KEY=VALUE` CLI flag (repeatable)
2. `SURREALKIT_VAR_<KEY>` environment variable
3. `[variables]` section in `surrealkit.toml`

```toml
# surrealkit.toml
[variables]
schema_prefix = "myapp"
talent_username = "talent_rw"
```

```bash
surrealkit sync --var schema_prefix=acme --var talent_username=talent_rw
```

- An **undefined variable is a hard error** — SurrealKit never silently skips it or leaves the token in the SQL.
- Escape a literal `${...}` by doubling the dollar sign: `$${literal}`.
- Substitution runs on `sync`, `seed`, `apply`, and `rollout start/complete/rollback`. It does **not** run on `rollout plan/baseline/status/lint` (no user SQL executes there).

## Project layout

`surrealkit init` creates a `database/` directory (override the root with `--folder` / `SURREALDB_FOLDER`):

```
database/
├── schema/                     # Schema definitions (.surql) — the source of truth
├── rollouts/                   # Generated rollout manifests (.toml)
├── snapshots/                  # Internal drift tracking
│   ├── schema_snapshot.json
│   └── catalog_snapshot.json
├── seed/                       # Seeding files (.surql)
├── tests/
│   ├── suites/                 # Test suites (.toml)
│   ├── fixtures/               # Test fixture data (.surql)
│   └── config.toml             # Global test config
└── setup.surql                 # One-time setup script
surrealkit.toml                 # Project config ([variables], [typegen])
```

## Rules & conventions

- **Sync vs rollout:** use `sync` for local, preview, and other disposable databases where it is safe to match files immediately; use `rollout` for shared/production databases that need review, staged execution, rollback, or operator-controlled cutover.
- Schema files are the source of truth. `sync` creates, updates, and prunes SurrealKit-managed objects to match `database/schema/`.
- Schema files should contain `DEFINE`/`REMOVE` statements. Allow other statements (`INSERT`, `UPDATE`, `CREATE`) only with `--allow-all-statements`, which disables catalog entity tracking.
- Store SurrealQL in files with the `.surql` extension. Validate and format generated SurrealQL with the tools described in the **surrealql** skill (`surreal validate`, `npx @surrealdb/surql-fmt`).
- SurrealKit is young and evolving; confirm command surfaces against `surrealkit --help` and the [README](https://github.com/surrealdb/surrealkit).

## References

- Project scaffolding and templates — [references/init-templates.md](references/init-templates.md)
- Development sync and production rollouts — [references/sync-rollouts.md](references/sync-rollouts.md)
- Type generation (JSON and TypeScript) — [references/typegen.md](references/typegen.md)
- Declarative testing framework — [references/testing.md](references/testing.md)
