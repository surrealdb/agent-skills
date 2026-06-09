---
title: Type Generation
---

# Type Generation

`surrealkit typegen` introspects a **live database** and emits a structured
schema document describing tables, fields, functions, and params. JSON is the
default output; TypeScript is an optional, config-driven emitter.

Because typegen reads the live database, run `surrealkit sync` (or apply your
rollout) first so the database reflects your current schema.

## JSON output

```bash
surrealkit typegen                 # writes {folder}/types/schema.json (pretty)
surrealkit typegen --out path.json # custom output path
surrealkit typegen --stdout        # print JSON to stdout instead of a file
surrealkit typegen --compact       # single-line JSON instead of pretty-printed
```

| Flag | Behaviour |
| --- | --- |
| `--out <path>` | Output path (default `{folder}/types/schema.json`) |
| `--stdout` | Print to stdout instead of writing a file |
| `--compact` | Emit compact (single-line) JSON |

The default folder follows `--folder` / `SURREALDB_FOLDER` (so
`database/types/schema.json` by default).

## TypeScript output

TypeScript generation is **not a CLI flag** — it is enabled via the `[typegen]`
section of `surrealkit.toml`. When `typescript` is set, both
`surrealkit typegen` and `surrealkit sync --watch` write an `index.ts` of
SurrealDB JS SDK v2 interfaces into that directory.

```toml
# surrealkit.toml
[typegen]
typescript = "../src/types"            # directory for generated index.ts
format     = "biome check --write"     # optional formatter, e.g. prettier --write / eslint --fix
```

- One `interface` is generated per table, with `id: RecordId<'table'>` and
  fields mapped to TypeScript types (record links become `RecordId<'…'>`
  unions, optional fields become `| undefined`).
- The `format` command is run on the written file (the file path is appended
  as the final argument, e.g. `biome check --write <path>`). It inherits the
  working directory so it picks up your project's own config. A missing or
  failing formatter is a non-fatal warning — it never breaks typegen or sync.

### Regenerating on schema change

Because `sync --watch` regenerates types when `[typegen] typescript` is set,
the common dev loop is:

```bash
surrealkit sync --watch
```

Edit `database/schema/*.surql` → sync applies the change → `index.ts` is
rewritten automatically.

## With Vite

Use [`vite-plugin-surrealkit`](https://github.com/surrealdb/surrealkit/tree/main/packages/vite-plugin-surrealkit)
to run `surrealkit sync` (and thus typegen, when configured) from your Vite dev
server, avoiding a separate watch process:

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import { surrealkitPlugin } from 'vite-plugin-surrealkit';

export default defineConfig({
  plugins: [surrealkitPlugin()],
});
```

Install it with `npm i -D vite-plugin-surrealkit`. It runs sync on dev-server
startup, watches `database/schema/**/*.surql`, and re-runs on change. Options
include `syncArgs`, `schemaGlobs`, `runOnStartup`, `reloadOnSync`, `debounceMs`,
`logLevel`, and `failBuildOnError`.

## Workflow notes

- Regenerate types after schema changes and in CI to catch drift.
- Decide deliberately whether to commit generated types (reproducible, no build
  dependency on a live DB) or gitignore them (always fresh). Generated files
  carry a "Run `surrealkit typegen` to regenerate." header.
