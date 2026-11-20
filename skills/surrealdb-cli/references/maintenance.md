---
title: Maintenance & Operations
---

# Maintenance & Operations

Operational commands for versioning, upgrades, readiness checks, storage
repair, and SurrealML models. These commands are version-sensitive — confirm
flags with `surreal db <command> --help` for your installed binary. Use
`surreal +4.0.1 db <command> --help` to see another version's surface.

## version

```bash
surreal db version
```

Prints the version of the db component that runs (the launcher's own version is
`surreal --version`). Combined with an endpoint it also reports the
remote server version:

```bash
surreal db version --endpoint http://localhost:8000
```

## is-ready

Returns exit code `0` once the server accepts connections and non-zero
otherwise — ideal for CI gates, container healthchecks, and startup scripts.
The alias `surreal db isready` also works.

```bash
surreal db is-ready --endpoint http://localhost:8000

# Wait for the server in a script
until surreal db is-ready -e http://localhost:8000 2>/dev/null; do
  sleep 1
done
echo "SurrealDB is up"
```

## upgrade & managing versions

The launcher manages the CLI and every component's versions. `surreal db upgrade`
is not run (the `upgrade` subcommand was removed from `db` in 4.x and refused
by the launcher in 3.x) — use `surreal upgrade` instead.

| Command | Effect |
| --- | --- |
| `surreal list` | Show each component, its default version, all installed versions, and paths (`--json` available) |
| `surreal install db@4.0.1` | Install another version beside the others (`--pin` makes it the default) |
| `surreal use db@4.0.1` | Make an installed version the default (never downloads); `db@4` picks the newest installed 4.x |
| `surreal upgrade` | Upgrade the launcher and every installed component |
| `surreal upgrade db` | Move the db to the newest release on its channel |
| `surreal upgrade db@3.3.5` | Move the db to an exact version |
| `surreal uninstall db@3.3.0` | Remove one version (`db` removes all) |
| `surreal upgrade --dry-run` | Preview without changing anything |

`surreal upgrade --version 3.3.5` / `--beta` / `--alpha` / `--nightly` are
shortcuts for `db@<VERSION>` / `db@beta` / etc.

If SurrealDB was installed via a package manager (Homebrew) or runs in Docker,
upgrade through that tool instead.

## fix

After a major upgrade, an on-disk store may use an older storage format. `fix`
migrates it in place. Point it at the same `PATH` used by `surreal db start`, and
run it while the server is stopped.

```bash
surreal db fix rocksdb:mydata.db
```

## ml — SurrealML models

> Available in db 3.x only. The 4.x `db` component has no `ml` subcommand; run
> it with `surreal +3.3.5 db ml …` (any installed 3.x version).

`surreal db ml` imports and exports [SurrealML](https://surrealdb.com/docs/surrealml)
models. Both subcommands take the standard connection flags plus `--namespace`
and `--database`; export also requires `--name` and `--version`.

```bash
# Import a model file
surreal db ml import --endpoint http://localhost:8000 \
  --user root --pass root --ns main --db main model.surml

# Export a specific model + version
surreal db ml export --endpoint http://localhost:8000 \
  --user root --pass root --ns main --db main \
  --name my_model --version 0.1.0 model.surml
```

Models are scoped to a namespace and database, so target the same `--ns`/`--db`
on import and export.
