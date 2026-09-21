---
title: Static Analysis
---

# Static Analysis

`surrealkit check`, `generate`, and `watch` run the
[SurrealQL Analyzer](https://github.com/surrealdb/analyzer) over the project.
**No database is contacted**, and none of these commands needs the environment
one would — a project whose `[target.*]` reads its password from an unset
variable still checks. That is what lets a pull-request job run
`surrealkit check --json` with no credentials at all.

The analyzer reads:

- every module's `schema/` directory as **the schema**
- every other `.surql` file as **queries**
- SurrealQL embedded in host code — `db.query("SELECT …")` in `.ts`, `.svelte`,
  `.vue`, `.astro`

and reports contract violations before anything reaches an instance: unknown
tables and fields, kind mismatches, bad graph traversals, comparisons that can
never be true, clauses the engine accepts and then ignores.

```bash
surrealkit check                                     # rustc-style findings; exit 1 on any error
surrealkit check --json                              # { summary, diagnostics[] } for CI and tooling
surrealkit check --watch                             # re-check on every save
surrealkit generate --out src/lib/db.generated.ts    # typed client for the embedded queries
surrealkit watch                                     # check, then regenerate, on every save
surrealkit watch --check-only                        # check loop with no writes (rejects --out)
```

## Output streams

Findings and the summary go to **stderr**, the way `rustc` and `tsc` report
them, so `surrealkit check > log.txt` cannot be what hides an error. `--json` is
the exception: it is the product of the run and goes to stdout alone, so
`surrealkit check --json | jq` reads one document and nothing else. In that
document `source` is the file relative to the project root, and `range` is a
pair of byte offsets into that file.

## Scope

Analysis **always covers the whole project**. `-s/--schema` picks which
directories are read *as schema* — and so are analyzed before the queries that
reference them — but every `.surql` and host file under the project root is read
either way, and a finding is reported wherever it lands. `--target` and `--all`
do not apply and are reported as ignored.

## Configuration

Schema directories come from the module layout, so nothing is written down twice.

```toml
# surrealkit.toml
[analyze]
# The SurrealDB release you deploy to. Turns on the version checks (a function
# or syntax the release lacks or removed). Unset means the latest release.
surrealdb_version = "3.2"

# Where `surrealkit generate` writes the typed client, relative to the directory
# holding surrealkit.toml. (`--out` on the command line is relative to the
# working directory, like any path typed at a shell.)
out = "src/lib/db.generated.ts"

# Extra directories to skip; target/, node_modules/ and .git/ always are.
ignore = ["dist/**"]

warnings_as_errors = false          # a CI gate

[analyze.lints]                     # "allow" | "warn" | "deny", by code or family
"7xxx" = "warn"
E1002 = "allow"
```

`ignore` entries are **directory names**, matched against every path component —
not globs. `dist` and `dist/**` are the same pattern; `src/generated/**` and
`*.gen.ts` match nothing. A pattern that would swallow a module's schema
directory (`"schema"`, `"database"`) is rejected rather than silently emptying
the analysis.

Suppress a finding inline with
`-- surrealql-analyzer: allow(E1001) reason="…"`. The full code catalog is at
<https://surrealguard.dev/docs/diagnostics>.

## Generation safety

`generate` refuses to overwrite a good registry when an embedded query has an
error, and `watch` regenerates only after a clean check, so the generated types
never lag behind a broken save.

A watch re-reads `surrealkit.toml` before every run, so editing the target
version or the lint levels re-targets the next one. `[analyze] out` is the
exception: it is read once when the watch starts, because the path the watcher
keeps out of its own input set has to stay fixed for the life of the process.

## Availability

These three commands only exist when the binary is built with the `analyze`
feature, which is **on by default** and included in prebuilt binaries and
`cargo binstall`. A build made with
`cargo install surrealkit --no-default-features --features kv-mem,cli` has no
`check`, `generate`, or `watch` — it still reads an `[analyze]` section without
complaint, so check `surrealkit --help` if the subcommands appear to be missing.

## Relationship to `typegen`

`check` / `generate` are static: schema files in, types and diagnostics out, no
database. `typegen` introspects a **live** database — see
[typegen.md](typegen.md). They solve different problems and can be used
together.
