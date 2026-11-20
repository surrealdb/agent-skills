---
name: surrealdb-cli
description: "Use the `surreal` command-line binary to run a SurrealDB server (in-memory, RocksDB, SurrealKV, or TiKV), open an interactive or piped SQL REPL, import and export databases, check server readiness, install, switch and pin multiple database versions (`surreal +4.0.1 db …`), repair storage, and manage SurrealML models. Use this skill whenever users run, connect to, back up, restore, or operate SurrealDB from the terminal. Triggers: surreal db start, surreal db sql, surreal db import, surreal db export, surreal install, surreal cloud, surreal memory, surreal use, surreal list, surreal upgrade, surreal +VERSION, surreal fix, surreal ml, SurrealDB CLI, surreal server."
metadata:
  author: surrealdb
  version: "0.2.0"
---

# SurrealDB CLI

A skill for driving the [`surreal`](https://surrealdb.com/docs/surrealdb/cli) command-line binary — SurrealDB's all-in-one launcher for running a server, querying it, moving data in and out, and installing multiple database versions side by side.

This skill covers operating the `surreal` binary itself. To write the SurrealQL that you run with it, use the **surrealql** skill; for declarative schema migrations and type generation, use the **surrealkit** skill.

## When to use this skill

Reference these guidelines when:

- Starting a SurrealDB server, locally or in a container (`surreal db start`)
- Opening the SQL shell or piping queries from a file (`surreal db sql`)
- Importing or exporting a database for backups, seeding, or migration (`surreal db import` / `surreal db export`)
- Gating scripts or CI on server availability (`surreal db is-ready`)
- Installing, switching, pinning or running a specific database version (`surreal install`, `surreal use`, `surreal +4.0.1 db …`)
- Installing or using the `cloud`, `memory`, `kit`, `lsp`, `ent` or `ext` components (`surreal install cloud`, `surreal memory --help`)
- Upgrading the CLI or database, or repairing on-disk storage after an upgrade (`surreal upgrade` / `surreal db fix`)
- Importing or exporting SurrealML models (`surreal db ml`, db 3.x only)

## Installation

| Method | Command |
| --- | --- |
| curl (macOS / Linux) | `curl -sSf https://install.surrealdb.com \| sh` |
| Windows (PowerShell) | `iwr https://windows.surrealdb.com -useb \| iex` |
| Homebrew | `brew install surrealdb/tap/surreal` |
| Docker | `docker run --rm --pull always -p 8000:8000 surrealdb/surrealdb:latest start` |

Verify with `surreal --version` (launcher) and `surreal list` (installed components and versions). Pin a specific image tag (not `latest`) for production. See the [install docs](https://surrealdb.com/install) for the current set of methods.

## Components & versions

`surreal` is a launcher (v1.x) that dispatches to separately installed
components: `db` (the database), `ent`, `memory`, `cloud`, `ext`, `kit`, `lsp`.
Server commands live under `db`: `surreal db start`, not `surreal start`. The
bare forms (`surreal start`, `surreal sql`, `surreal version`, …) still run but
are deprecated and print a warning.

Several `db` versions can be installed at once. Prefix any command with
`+VERSION` to run that version for just that command:

```bash
surreal list                          # components, default version, installed versions
surreal install db@4.0.1              # install beside existing versions
surreal +4.0.1 db start --user root --pass root memory   # run 4.0.1 once
surreal +3.3.5 db export --ns main --db main > dump.surql
surreal use db@4.0.1                  # make 4.0.1 the default (no download)
```

`+VERSION` must already be installed; otherwise the CLI tells you to run
`surreal install db@VERSION`. Use `+VERSION` to try a new major against a copy
of your data, or to export from an old server and import into a new one. See
[maintenance.md](references/maintenance.md) for `install`, `use`, `upgrade` and
`uninstall`.

The `db` surface differs by version — 4.x has no `ml`, `module`, `upgrade` or
`v2` subcommands. Check `surreal +VERSION db --help`.

### Other components

Besides `db`, the launcher fronts these components. They are **not installed by
default**: running one that is missing fails with an install hint (it will not
download without confirmation). Install on demand, and learn each one's
arguments from its own `--help` rather than guessing — their surface changes
independently of `db`.

| Component | Purpose | Install |
| --- | --- | --- |
| `cloud` | SurrealDB Cloud: organisations, instances, backups, teams | `surreal install cloud` |
| `memory` | Agent memory: remember, recall, reflect | `surreal install memory` |
| `kit` | Schema migrations, seed data, typegen, tests (see the **surrealkit** skill) | `surreal install kit` |
| `lsp` | SurrealQL language server for editors (see the **surrealql-functions** skill) | `surreal install lsp` |
| `ent` | SurrealDB Enterprise, the licensed edition of the database | `surreal install ent` |
| `ext` | Build and run WebAssembly extensions | `surreal install ext` |

```bash
surreal list                    # which components are installed
surreal install cloud           # install (add -y to skip the prompt)
surreal cloud --help            # discover subcommands and flags
surreal memory --help
```

When a task needs one of these (managing a Cloud instance, storing or recalling
agent memory), check `surreal list`, install the component if the user's task
calls for it, then use `surreal <component> --help` and
`surreal <component> <subcommand> --help` to work out the arguments. Both
`cloud` and `memory` act on remote accounts or data, so confirm destructive
operations (deleting instances, forgetting memories) with the user first.

## Command map

| Command | Purpose | Reference |
| --- | --- | --- |
| `surreal db start [PATH]` | Run the database server with a chosen storage backend | [start.md](references/start.md) |
| `surreal db sql` | Interactive REPL or piped, non-interactive querying | [sql.md](references/sql.md) |
| `surreal db import <FILE>` | Load a SurrealQL script into a database | [import-export.md](references/import-export.md) |
| `surreal db export [FILE]` | Dump a database to SurrealQL (file or stdout) | [import-export.md](references/import-export.md) |
| `surreal db is-ready` | Exit 0 when the server is accepting connections (alias `isready`) | [maintenance.md](references/maintenance.md) |
| `surreal db version` | Print the db version and remote server version | [maintenance.md](references/maintenance.md) |
| `surreal list` / `install` / `use` / `uninstall` | Manage installed component versions | [maintenance.md](references/maintenance.md) |
| `surreal upgrade [COMPONENT[@VERSION]]` | Upgrade the launcher and components | [maintenance.md](references/maintenance.md) |
| `surreal db fix [PATH]` | Migrate on-disk storage to the current format | [maintenance.md](references/maintenance.md) |
| `surreal db ml <sub>` | Import / export SurrealML models (db 3.x only) | [maintenance.md](references/maintenance.md) |

`surreal db validate` (SurrealQL file checking) and formatting are covered by the **surrealql** skill. `surreal db mcp` (MCP server over stdio) also exists — run `surreal db --help` to confirm the surface of the installed version. Other components are covered under "Other components" above.

## Connection flags & env vars

`db sql`, `db import`, `db export`, and `db ml` share the same connection flags. Each flag has a `SURREAL_*` environment variable fallback; an explicit flag wins.

| Flag (aliases) | Env var | Notes |
| --- | --- | --- |
| `-e, --endpoint` | — | `ws://localhost:8000` for `sql`; `http://localhost:8000` for `import`/`export` |
| `--namespace` (`--ns`) | `SURREAL_NAMESPACE` | Required for `import`/`export`/`ml` |
| `--database` (`--db`) | `SURREAL_DATABASE` | Required for `import`/`export`/`ml` |
| `-u, --username` (`--user`) | `SURREAL_USER` | |
| `-p, --password` (`--pass`) | `SURREAL_PASS` | |
| `-t, --token` | `SURREAL_TOKEN` | JWT; use instead of username/password |
| `--auth-level` | `SURREAL_AUTH_LEVEL` | `root` (default), `namespace`/`ns`, or `database`/`db` |

Pass secrets via env vars rather than flags so they do not leak into shell history or process listings.

## Quick start

```bash
# 1. Start an authenticated in-memory server (foreground)
surreal db start --user root --pass root memory

# 2. In another shell, open the SQL REPL on namespace/database "main"
surreal db sql --endpoint ws://localhost:8000 \
  --user root --pass root --ns main --db main --pretty

# 3. Load a dump into that database
surreal db import --endpoint http://localhost:8000 \
  --user root --pass root --ns main --db main backup.surql
```

## Rules & conventions

- **Authentication is on by default.** `surreal db start` needs `--user`/`--pass`; only use `--unauthenticated` for throwaway local instances.
- **Choose a storage backend deliberately.** `memory` (the default) is ephemeral — data vanishes on exit. Use `rocksdb:`/`surrealkv://` for persistence and `tikv://` for distributed clusters. See [start.md](references/start.md).
- **Export defaults to stdout.** `surreal db export` writes to `-` (stdout) unless you give a file path; redirect or pipe accordingly.
- **Gate CI with `is-ready`.** It returns a non-zero exit code until the server accepts connections — ideal in healthchecks and scripts.
- **Run `surreal db fix` after a major upgrade** if the server reports an incompatible storage version, pointing it at the same `PATH` used by `start`.
- **Pin the version for anything that matters.** Use `+VERSION` (or `surreal use`) so scripts do not change behaviour when the default moves; back up with `export` before running a new major against existing data.
- **The CLI evolves.** Confirm flags and subcommands against `surreal db <command> --help` (or `surreal +VERSION db …`); this skill reflects launcher 1.2 with db 3.3 / 4.0.

## References

- Running a server and storage backends — [references/start.md](references/start.md)
- Interactive and piped querying — [references/sql.md](references/sql.md)
- Importing and exporting data — [references/import-export.md](references/import-export.md)
- Versioning, upgrades, readiness, repair, and ML models — [references/maintenance.md](references/maintenance.md)
