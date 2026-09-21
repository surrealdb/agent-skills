---
name: surrealkit
description: "Use SurrealKit, SurrealDB's schema-management and migration CLI, to scaffold projects from templates, sync schema in development, plan and execute production rollouts (with rollback), statically check SurrealQL without a database, generate JSON/TypeScript types, and write declarative TOML test suites for schemas, permissions, and API endpoints. Use this skill whenever users set up, migrate, type, check, or test a SurrealDB schema with SurrealKit."
metadata:
  author: surrealdb
  version: "0.2.0"
---

# SurrealKit

A skill for driving [SurrealKit](https://github.com/surrealdb/surrealkit), SurrealDB's schema-management and migration CLI. Written against **SurrealKit 1.0** (currently `1.0.0-beta.3`).

SurrealKit keeps a SurrealDB database in sync with `.surql` schema files. It provides two complementary workflows — a fast declarative **sync** for development and controlled, phased **rollouts** for shared and production databases — plus seeding, static analysis, type generation, and a declarative testing framework.

## When to use this skill

Reference these guidelines when:

- Scaffolding a new SurrealDB project (`surrealkit init`) or authoring templates
- Applying schema changes in development (`surrealkit sync`)
- Planning, executing, or rolling back production migrations (`surrealkit rollout`)
- Adopting SurrealKit on a database that already exists (`surrealkit rollout baseline`)
- Splitting a schema into independently tracked modules, or applying it to several databases
- Statically checking SurrealQL against the schema without a database (`surrealkit check`)
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

Building from source needs a C compiler and `cmake`. Static analysis is on by default and adds `tree-sitter` (~7.5 MiB, ~100s of build time); drop it with `cargo install surrealkit --no-default-features --features kv-mem,cli` — that binary has no `check`, `generate`, or `watch`. Prebuilt binaries and `cargo binstall` always include analysis.

The Docker image is distroless (~25 MB, uid 65532) and exits on completion, which suits "apply schema then run tests" pipelines in Docker Compose alongside SurrealDB.

## Command map

| Command | Purpose | Reference |
| --- | --- | --- |
| `surrealkit init` | Scaffold a project from a template, selecting optional features | [init-templates.md](references/init-templates.md) |
| `surrealkit sync` | Declaratively reconcile the database to your schema files (dev) | [sync-rollouts.md](references/sync-rollouts.md) |
| `surrealkit rollout <sub>` | Plan, stage, complete, roll back, and repair migrations (shared/prod) | [sync-rollouts.md](references/sync-rollouts.md) |
| `surrealkit seed [--force]` | Run seeding files in `database/seed/` (idempotent, hash-tracked) | [sync-rollouts.md](references/sync-rollouts.md) |
| `surrealkit check` | Statically analyse the project's SurrealQL — no database contacted | [static-analysis.md](references/static-analysis.md) |
| `surrealkit generate` / `watch` | Emit / continuously refresh a typed client for embedded queries | [static-analysis.md](references/static-analysis.md) |
| `surrealkit typegen` | Introspect a live DB and emit JSON / TypeScript types | [typegen.md](references/typegen.md) |
| `surrealkit test` | Run declarative TOML test suites | [testing.md](references/testing.md) |
| `surrealkit apply <path>` | Apply a single `.surql` file directly | — |
| `surrealkit setup` | Run `database/setup.surql` (SurrealKit's own metadata tables) | — |
| `surrealkit status` | Show sync/rollout state | — |

Run `surrealkit <command> --help` to confirm available flags for an installed version.

## Connection & config

Global flags work on every command and resolve in this order (highest wins):
**`[target.<name>]` in `surrealkit.toml` > CLI flags > system env vars > `.env` file > defaults**.

```bash
surrealkit --host http://localhost:8000 --ns my_ns --db my_db \
  --user root --pass root --auth-level root sync
```

| Flag | Env var | Default |
| --- | --- | --- |
| `--host` | `SURREALDB_HOST` | `http://localhost:8000` |
| `--ns` | `SURREALDB_NAMESPACE` | `db` |
| `--db` | `SURREALDB_NAME` | `test` |
| `--user` | `SURREALDB_USER` | `root` |
| `--pass` | `SURREALDB_PASSWORD` | `root` |
| `--auth-level` | `SURREALDB_AUTH_LEVEL` | `root` (`root` / `namespace`\|`ns` / `database`\|`db` / `none`) |
| `--folder` | `SURREALDB_FOLDER` | `./database` |
| `--connect-timeout-secs` | `SURREALDB_CONNECT_TIMEOUT_SECS` | `30` (`0` waits indefinitely) |
| `--query-timeout-secs` | `SURREALDB_QUERY_TIMEOUT_SECS` | unset (a rollout step's SQL waits indefinitely) |

> **The `DATABASE_*` aliases were removed in 1.0.** Setting one without its `SURREALDB_*` replacement is a hard error, not a silent fallback. Update `.env` files and CI when upgrading.

Selection flags are also global: `-s/--schema <name>`, `-t/--target <name>`, `--all`, `--keep-going`, `--no-deps` — see [modules-targets.md](references/modules-targets.md).

The project root holds `surrealkit.toml` with `[variables]`, `[typegen]`, `[analyze]`, `[schema.*]`, and `[target.*]` sections.

### Embedded databases

Point `--host` at an embedded endpoint to manage an in-process datastore; authentication is skipped automatically.

```bash
surrealkit --host surrealkv://./data --ns my_ns --db my_db sync
```

Schemes: `surrealkv://`, `rocksdb://`, `speedb://`, `mem://`, `file://`, `tikv://`. `--auth-level none` forces the no-signin path on any endpoint. The **prebuilt CLI bundles only the in-memory engine** — for on-disk engines build with `--features kv-surrealkv` (or `kv-rocksdb`, or `embedded`). Embedded engines take an exclusive lock, so stop the application before running the CLI against one; to manage schema from inside an application at startup, use the library instead.

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
- Substitution is textual, so `${VAR}` inside a SurrealQL string literal is also replaced.
- Substitution runs on `sync`, `seed`, `apply`, and `rollout start/complete/rollback`. It does **not** run on `rollout plan/baseline/status/lint` (no user SQL executes there).

## Project layout

`surrealkit init` creates a `database/` directory (override the root with `--folder` / `SURREALDB_FOLDER`):

```
database/
├── schema/                     # Schema definitions (.surql) — the default module, the source of truth
├── modules/<name>/schema/      # Additional schema modules, when declared in surrealkit.toml
├── rollouts/                   # Generated rollout manifests (.toml)
├── snapshots/                  # Drift tracking — what `rollout plan` diffs against
│   ├── schema_snapshot.json
│   └── catalog_snapshot.json
├── seed/                       # Seeding files (.surql)
├── tests/
│   ├── suites/                 # Test suites (.toml)
│   ├── fixtures/               # Test fixture data (.surql)
│   └── config.toml             # Global test config
└── setup.surql                 # One-time setup script (SurrealKit's own metadata tables)
surrealkit.toml                 # Project config
```

SurrealKit tracks its own state in three metadata tables on your database: `__entity` (file hashes and per-definition records), `__rollout` (rollout run state), and `__seed` (seed-file hashes).

## Rules & conventions

- **Sync vs rollout:** use `sync` for local, preview, and other disposable databases where it is safe to match files immediately; use `rollout` for shared/production databases that need review, staged execution, rollback, or operator-controlled cutover.
- Schema files are the source of truth. `sync` creates, updates, and prunes SurrealKit-managed objects to match the schema directory.
- Schema files should contain `DEFINE`/`REMOVE` statements. Allow other statements (`INSERT`, `UPDATE`, `CREATE`) only with `--allow-all-statements`, which disables catalog entity tracking.
- **`rollout plan` diffs against `database/snapshots/`, not the live database, and `sync` never writes snapshots.** Adopting rollouts after a period of sync-only work needs a `rollout baseline` first, or the first plan proposes every file you have.
- Commit the generated manifest *and* the refreshed snapshots together; reverting a plan means reverting both.
- Store SurrealQL in files with the `.surql` extension. Validate and format generated SurrealQL with the tools described in the **surrealql** skill (`surreal validate`, `npx @surrealdb/surql-fmt`), and gate it in CI with `surrealkit check`.
- SurrealKit is young and evolving; confirm command surfaces against `surrealkit --help` and the [README](https://github.com/surrealdb/surrealkit).

## Using SurrealKit as a Rust library

The `surrealkit` crate runs sync, seed, and rollouts from inside an application — useful for applying schema at startup, or for embedded engines the CLI cannot hold a lock on.

```toml
[dependencies]
surrealkit = { version = "1.0.0-beta.3", default-features = false, features = ["kv-mem"] }
```

- `default = ["kv-mem", "cli"]`; library consumers drop `cli` to shed `clap`, `inquire`, `rustls`, and `tempfile`.
- `embed_schema!` and `embed_seed!` compile the schema and seed files into the binary, so no filesystem is needed at runtime.
- `Sync` is a builder for runtime control; `Rollout` no longer writes to disk as of 1.0.
- Progress is emitted through the [`log`](https://docs.rs/log) facade; without a logger the library is silent.

Full API reference: [`crates/surrealkit/README.md`](https://github.com/surrealdb/surrealkit/blob/main/crates/surrealkit/README.md). Upgrade notes: [`MIGRATING.md`](https://github.com/surrealdb/surrealkit/blob/main/MIGRATING.md).

## References

- Project scaffolding and templates — [references/init-templates.md](references/init-templates.md)
- Development sync, production rollouts, and seeding — [references/sync-rollouts.md](references/sync-rollouts.md)
- Schema modules and database targets — [references/modules-targets.md](references/modules-targets.md)
- Static analysis (`check` / `generate` / `watch`) — [references/static-analysis.md](references/static-analysis.md)
- Type generation (JSON and TypeScript) — [references/typegen.md](references/typegen.md)
- Declarative testing framework — [references/testing.md](references/testing.md)
