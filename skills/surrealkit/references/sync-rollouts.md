---
title: Sync & Rollouts
---

# Sync & Rollouts

SurrealKit separates schema authoring from how changes reach a database:

- **`sync`** — a fast desired-state reconciler for local, preview, and other
  disposable databases. Edit `database/schema/*.surql`, run sync, and the
  database is made to match.
- **`rollout`** — a controlled, phased migration path for shared and production
  databases: changes are planned into reviewed manifests, applied in stages, and
  can be rolled back.

Use `sync` when it is safe for the database to match local files immediately.
Use `rollout` when changes need review, staged execution, rollback, or
operator-controlled cutover.

## Sync (development)

```bash
surrealkit sync                 # reconcile once
surrealkit sync --watch         # re-sync on file changes, incl. deletions
surrealkit sync --dry-run       # show what would change without applying
```

`sync` applies changed schema files and automatically removes
SurrealKit-managed objects that were deleted from `database/schema`.

| Flag | Behaviour |
| --- | --- |
| `--watch` | Watch schema files and re-sync on change |
| `--debounce-ms <n>` | Debounce window for watch (default `1000`) |
| `--dry-run` | Report changes without applying them |
| `--fail-fast` | Stop on first error (default `true`) |
| `--no-prune` | Do not remove objects deleted from schema files |
| `--allow-shared-prune` | Required override to allow destructive prune against a shared DB |
| `--allow-all-statements` | Permit non-`DEFINE` statements (e.g. `INSERT`, `UPDATE`, `CREATE`); disables catalog entity tracking — only file-level hashes are tracked |

### Hash-based re-sync gotcha

`sync` tracks schema files by **content hash**. Changing a template variable's
*value* does not change the file's hash, so sync will not re-apply that file.
To force re-application, touch the file or remove its tracking entry. In watch
mode, variables are resolved once at startup; edits to `surrealkit.toml`
require a restart.

## Rollouts (shared / production)

The rollout lifecycle follows an expand → cutover → contract pattern:

```bash
# 1. Baseline an existing shared/prod DB before the first rollout
surrealkit rollout baseline

# 2. Generate a manifest from the current desired-state diff
surrealkit rollout plan --name add_customer_indexes
#    → writes database/rollouts/<timestamp>__add_customer_indexes.toml

# 3. Apply the non-destructive expansion phase
surrealkit rollout start 20260302153045__add_customer_indexes

# 4. Let application cutover happen, then run the destructive contract phase
surrealkit rollout complete 20260302153045__add_customer_indexes
```

| Subcommand | Purpose |
| --- | --- |
| `baseline` | Establish initial state on an existing shared/prod DB |
| `plan --name <n> [--dry-run]` | Turn the desired-state diff into a reviewed manifest |
| `start <target>` | Apply the non-destructive expansion phase; records resumable state |
| `complete <target>` | Perform the destructive contract phase (e.g. remove legacy objects) after cutover |
| `rollback <target>` | Revert an in-flight rollout |
| `lint <target>` | Validate a manifest without mutating the database |
| `status [target]` | Inspect rollout state stored in the database |
| `repair <target>` | Heal a rollout stuck in an intermediate state (metadata only, no SQL) |

Generated manifests live in `database/rollouts/*.toml`. Local snapshots are
tracked in `database/snapshots/schema_snapshot.json` and `catalog_snapshot.json`.

### Recovering a stuck rollout

If `complete` or `rollback` is killed mid-flight, the `__rollout` row can be
left in an intermediate state (`running_complete`, `running_rollback`, or
`running_start`) even though the schema is already materialised. Re-running
`complete`/`rollback` will not always heal the metadata because the SQL steps
are already applied.

```bash
surrealkit rollout repair 20260302153045__add_customer_indexes
```

Behaviour by stuck state:

- `running_complete` → flips to `completed`, restores `target_entities`.
- `running_rollback` → flips to `rolled_back`, restores `source_entities`.
- `running_start` → flips to `failed` with a note; re-run `start` (idempotent)
  or `rollback`.

`repair` never re-executes per-step SQL — it only reconciles `__rollout` and
`__entity` so subsequent `sync` / `plan` runs see a clean state.

## Template variables in sync/rollout

`--var KEY=VALUE` (repeatable) works on `sync`, `seed`, `apply`, and
`rollout start/complete/rollback`:

```bash
surrealkit sync --var schema_prefix=acme
surrealkit rollout start my_rollout --var schema_prefix=acme
```

Substitution does **not** run on `rollout plan`, `baseline`, `status`, or
`lint`, since they execute no user SQL. See the main SKILL.md for full variable
resolution rules. Entity names containing `${VAR}` tokens appear literally in
`catalog_snapshot.json` and are not substituted, which affects drift detection —
prefer fixed entity names in production schemas.

## Seeding

```bash
surrealkit seed
```

Runs the seeding files in `database/seed/` on demand. Template variables apply.

## Running sync from Vite

The [`vite-plugin-surrealkit`](https://github.com/surrealdb/surrealkit/tree/main/packages/vite-plugin-surrealkit)
package runs `surrealkit sync` from a Vite dev/build process so you do not need
a separate `surrealkit sync --watch` terminal. See [typegen.md](typegen.md) for
its use alongside TypeScript generation.
