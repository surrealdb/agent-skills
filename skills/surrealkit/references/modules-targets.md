---
title: Schema Modules & Database Targets
---

# Schema Modules & Database Targets

By default a project has one unnamed schema (`database/schema`) applied to one
database. That is still the default and needs no configuration — it is the
pre-1.0 layout.

For larger projects, declare **schema modules** (independently tracked sets of
schema files) and **targets** (named databases to apply them to).

```toml
# surrealkit.toml
[schema.core]

[schema.billing]
depends_on = ["core"]

[target.acme]
ns = "acme"
db = "prod"
primary = true

[target.globex]
ns = "globex"
db = "prod"
pass_env = "GLOBEX_DB_PASSWORD"   # never inline a password
```

```
database/
  schema/                    # the default module
  modules/
    core/schema/
    billing/schema/
```

Named modules live under `modules/<name>/` rather than inside `database/schema`,
because the default module walks its schema directory recursively — nesting
would make it collect the named module's files and claim ownership of them. Set
`[schema.<name>] path` for a custom location.

## Selection flags

These are global flags, available on every command that touches a database:

| Flag | Behaviour |
| --- | --- |
| `-s`, `--schema <name>` | Operate on this module (repeatable). Pulls in its `depends_on` modules |
| `-t`, `--target <name>` | Operate on this target (repeatable) |
| `--all` | Every declared module against every declared target |
| `--no-deps` | Do not pull in the dependencies of `--schema` selections |
| `--keep-going` | Continue to the next target after one fails |

```bash
surrealkit sync                              # every declared module, primary target
surrealkit sync --schema billing             # billing (and core, its dependency)
surrealkit sync --schema billing --no-deps   # billing alone
surrealkit sync --target acme                # every module, one target
surrealkit sync --all                        # the full matrix
surrealkit sync --all --keep-going           # don't stop at the first failure
```

Target resolution with no `--target`: the target marked `primary = true`, else
the single declared target if there is exactly one, else the ambient connection
from `--host`/`--ns`/`--db` and the environment.

## Why modules matter

Each module owns its own metadata, so **a module only ever prunes its own
database objects**. Without modules, syncing one schema against a database that
another schema had populated would remove the other's tables.

## Dependencies

`depends_on` orders application, so a module is never applied before what it
depends on. Selecting a module pulls its dependencies in, like `cargo build -p`;
`--no-deps` opts out. Cycles are rejected before anything touches a database.

## Targets and credentials

A target inherits everything it does not set from the ambient configuration
(`--host`/`--ns`/`--db` and the `SURREALDB_*` variables), so it usually only
needs `ns` and `db`.

Full key list for `[target.<name>]`: `host`, `ns`, `db`, `user`, `pass_env`,
`auth_level`, `schemas`, `primary`, `connect_timeout_secs`, `query_timeout_secs`.

- Passwords are read from the environment via `pass_env`. A literal `pass` or
  `password` key in `surrealkit.toml` is **rejected**, and an unset `pass_env`
  fails before any connection is opened rather than part-way through a fan-out.
- **A value set on a target wins over the equivalent CLI flag and environment
  variable** — for every key, not just the timeouts. Naming a target selects a
  described connection, so `--host` does not override a target's `host`. Omit a
  key to inherit (CLI flag → env var → `.env` → default).

Restrict which modules reach a target, and override deadlines for one that is
across a slower link:

```toml
[target.warehouse]
ns = "internal"
db = "analytics"
schemas = ["core", "analytics"]
connect_timeout_secs = 60   # default 30; 0 waits indefinitely
query_timeout_secs = 900    # default unset, meaning no deadline on step SQL
```

## Fan-out semantics

Targets are applied one at a time. Modules within a target stop at the first
failure, since they are dependency-ordered and the rest would build on a broken
base; targets also stop at the first failure unless you pass `--keep-going`.

There is no cross-database transaction, so a failed run can leave some targets
applied and others not. Every operation is idempotent, so re-running after a fix
is safe. `surrealkit sync --all` exits non-zero if any pair failed.

## Commands that cannot fan out

Rollout execution is a locked, resumable state machine with one `__rollout`
record per database, so `rollout start/complete/rollback/repair` refuse a
multi-target selection rather than leaving N databases in different phases —
pass a single `--target <name>`. Commands that operate on one module at a time
likewise require a single `--schema <name>`.

`check`, `generate`, and `watch` never connect, so `--target`/`--all` are
reported as ignored (a warning, not an error, so CI wrappers that pass the same
flags to every subcommand keep working).

## Empty-source preflight

Before opening any database connection, filesystem sync resolves every selected
module and **refuses if one has no `.surql` sources**. This prevents a wrong
working directory or `--folder` value from becoming setup or prune activity. Use
`--allow-empty-prune` only when an empty source set is intentional; a selection
where no module applies to any target remains an error rather than a successful
no-op.
