---
title: Sync, Rollouts & Seeding
---

# Sync, Rollouts & Seeding

SurrealKit separates schema authoring from how changes reach a database:

- **`sync`** — a fast desired-state reconciler for local, preview, and other
  disposable databases. Edit `database/schema/*.surql`, run sync, and the
  database is made to match.
- **`rollout`** — a controlled, phased migration path for shared and production
  databases: changes are planned into reviewed manifests, applied in stages, and
  can be rolled back.

Use `sync` when it is safe for the database to match local files immediately.
Use `rollout` when changes need review, staged execution, rollback, or
operator-controlled cutover. The connection flags are identical; what differs is
which database they point at and how much damage an accident does. Keep
shared-database credentials out of the `.env` used for local sync.

## Sync (development)

```bash
surrealkit sync                 # reconcile once
surrealkit sync --watch         # re-sync on file changes, incl. deletions
surrealkit sync --dry-run       # show what would change without applying
```

`sync` applies changed schema files and automatically removes
SurrealKit-managed objects that were deleted from the schema directory.

| Flag | Behaviour |
| --- | --- |
| `--watch` | Watch schema files and re-sync on change |
| `--debounce-ms <n>` | Debounce window for watch (default `1000`) |
| `--dry-run` | Report changes without applying them |
| `--fail-fast` | Stop on first error (default `true`) |
| `--no-prune` | Do not remove objects deleted from schema files |
| `--allow-shared-prune` | Required override to allow destructive prune against a shared DB |
| `--allow-empty-prune` | Allow sync to connect with no schema files — required before an intentional prune of every managed entity |
| `--allow-all-statements` | Permit non-`DEFINE` statements (e.g. `INSERT`, `UPDATE`, `CREATE`); disables catalog entity tracking — only file-level hashes are tracked |

Global selection flags (`-s/--schema`, `-t/--target`, `--all`, `--keep-going`,
`--no-deps`) apply too — see [modules-targets.md](modules-targets.md).

`--dry-run` prints which files would be applied. On an additive change that is
all you get; `REMOVE` statements only appear when something is stale — tracked
in `__entity` but no longer present in the schema files. Use it after deleting
or renaming a `DEFINE` to see the prunes before they land.

### Hash-based re-sync gotcha

`sync` tracks schema files by **content hash**. Changing a template variable's
*value* does not change the file's hash, so sync will not re-apply that file.
To force re-application, touch the file or remove its tracking entry. In watch
mode, variables are resolved once at startup; edits to `surrealkit.toml`
require a restart.

### Metadata tables

`database/setup.surql` creates three bookkeeping tables on your database. They
are SurrealKit's own state, not product schema:

| Table | Role |
| --- | --- |
| `__entity` | File hashes and one record per managed definition |
| `__rollout` | Rollout run state; empty until you use rollouts |
| `__seed` | Seed-file content hashes |

`__entity` records carry an `ns` discriminator:

- `ns: 'sync'` — one per schema file; `key` is the file path, `val.hash` its SHA-256. A matching hash on the next sync means the file's statements are skipped.
- `ns: 'schema'` — one per catalog object (`table::project`, `field:project:name`, indexes, …), with `val.source_path`, `val.file_hash`, `val.statement_hash` (the normalised statement, so a single changed field is detectable), and `val.state` (`active` means SurrealKit expects the object to exist).
- `ns: 'meta'` — sync bookkeeping such as `last_sync`.

These per-definition records are what let prune and rollouts reason about
individual tables and fields rather than whole files.

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
| `baseline` | Record the current state of an existing shared/prod DB as the diff origin. Reads only; writes the snapshot files locally |
| `plan --name <n> [--dry-run] [--allow-modified]` | Turn the desired-state diff into a reviewed manifest |
| `start <rollout-id>` | Apply the non-destructive expansion phase; records resumable state |
| `complete <rollout-id>` | Perform the destructive contract phase (e.g. remove legacy objects) after cutover |
| `rollback <rollout-id>` | Revert an in-flight rollout (only between `start` and `complete`) |
| `lint <rollout-id>` | Validate a manifest without mutating the database |
| `status [rollout-id]` | Inspect rollout state stored in the database |
| `repair <rollout-id>` | Heal a rollout stuck in an intermediate state (metadata only, no SQL) |

The **rollout id** is the manifest filename without `.toml` — e.g.
`database/rollouts/20260302153045__add_customer_indexes.toml` →
`20260302153045__add_customer_indexes`. The digits are a timestamp; the part
after `__` comes from `--name`. Run commands from the project root (the folder
containing `database/`).

### `plan` diffs snapshots, not the live database

`plan` compares your schema files against `database/snapshots/`, **not** against
the database — and `sync` never writes snapshots. A project that has been using
`sync` only has empty snapshots, so its first `plan` proposes every schema file
it has. Run `rollout baseline` first to record an agreed starting point; then
the next plan contains only what actually changed.

`plan` refreshes the snapshots on its way out, so commit the manifest and the
snapshots together. Reverting a plan you decided not to run means reverting
both.

A plan gets a `complete` phase only when it contains something destructive. A
purely additive plan has `start` and `rollback` phases only — and that
`rollback` step is the mirror of `start`, so on a first-plan-proposes-everything
manifest it would remove the whole schema.

### Modified entities: `--allow-modified`

`plan` handles additions and removals on its own. A change to an entity that
already exists — tightening an `ASSERT`, adjusting `PERMISSIONS` — needs
`--allow-modified`:

```bash
surrealkit rollout plan --name tighten_age_assert --allow-modified
```

The opt-in is about **rollback**, not about applying the change: applying it is
just `DEFINE ... OVERWRITE`. Undoing it means restoring the previous definition,
which `start` captures from the live database before the expand phase. That is a
clean undo for `ASSERT`, `PERMISSIONS`, and `COMMENT`, and only a partial one
for `TYPE`, `VALUE`, or `DEFAULT` changes, since it reverses the schema but not
data written under the new definition. Such a rollout is recorded as
`reversibility: definition_only`, and `rollout status` says so.

### State machine

States are recorded in the `__rollout` table:

```text
planned → running_start → ready_to_complete → running_complete → completed
                                   │
                                   └── running_rollback → rolled_back
```

| State | Meaning |
| --- | --- |
| `planned` | Manifest exists; neither start nor complete has run |
| `running_start` | Start phase in progress, or interrupted |
| `ready_to_complete` | Start phase completed successfully |
| `running_complete` | Complete phase in progress, or interrupted |
| `completed` | Both phases done (terminal; `rollback` refuses) |
| `running_rollback` | Rollback in progress, or interrupted |
| `rolled_back` | Start phase reversed (terminal) |
| `failed` | A phase failed; recover with `rollback` or `repair` |

Only one rollout may be in a non-terminal state at a time, so a stuck rollout
blocks the next one.

### Recovering a stuck rollout

Connect and sign-in are bounded by `--connect-timeout-secs` (30s by default), so
an endpoint that accepts the connection and never responds fails with a message
naming it rather than blocking. Step SQL is **not** bounded by default — a
legitimate index build can take hours — but every step logs when it starts and
every 15 seconds while it runs, so a slow step is distinguishable from a stuck
one. Bound it explicitly with `--query-timeout-secs` when you want to.

If `complete` or `rollback` is killed mid-flight, the `__rollout` row can be
left in an intermediate state even though the schema is already materialised.
Re-running `complete`/`rollback` will not always heal the metadata because the
SQL steps are already applied.

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

## Adopting SurrealKit on an existing database

1. `surrealkit init` in the repository root.
2. Mirror the live schema into `.surql` files under `database/schema/`. These
   become the target state, so they must reflect what is currently defined.
   `INFO FOR DB` / `INFO FOR TABLE` show what is there; this query returns every
   definition as an array of strings:

   ```surql
   LET $db = INFO FOR DB;
     $db.tables.values() +
     $db.users.values() +
     $db.tables.keys().map(|$t| {
       LET $i = INFO FOR TABLE $t;
       $i.fields.?.values() + $i.indexes.?.values()
     }).flatten().filter(|$v| !!$v);
   ```

3. `surrealkit rollout baseline` — reads the database, writes
   `database/snapshots/schema_snapshot.json` and `catalog_snapshot.json`. It
   does not modify the database.
4. From then on: edit schema files → `rollout plan` → `start` → deploy →
   `complete`.

`sync` also works against an existing database, but **it prunes definitions that
are not in your schema files**. If the files are incomplete, that deletes things
you did not mean to delete — which is why it requires `--allow-shared-prune`
against a database that already carries SurrealKit metadata. For production and
shared databases, use rollouts.

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
surrealkit seed           # applies new/changed seed files; skips unchanged ones
surrealkit seed --force   # re-run every seed file, ignoring the tracking table
```

Seeding runs on demand and is **idempotent**: each file in `database/seed/` is
tracked in the `__seed` table by content hash and runs only on first boot or
when its content changes. That makes it safe to run repeatedly, including on
every deploy. Template variables apply.

To ship seeds inside a binary with no filesystem at runtime, use the library's
[`embed_seed!`](https://github.com/surrealdb/surrealkit/blob/main/crates/surrealkit/README.md)
macro.

## Running sync from Vite

[`vite-plugin-surrealkit`](https://github.com/surrealdb/surrealkit/tree/main/packages/vite-plugin-surrealkit)
runs `surrealkit sync` from a Vite dev server or build, so you do not need a
separate `surrealkit sync --watch` terminal. Requires Vite 8. See
[typegen.md](typegen.md) for its use alongside TypeScript generation.
