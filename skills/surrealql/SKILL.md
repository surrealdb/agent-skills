---
name: surrealql
description: "Generate and modify SurrealQL queries to interact with SurrealDB databases. This includes creating and retrieving records, designing and managing schemas, establishing and querying graph relationships, performing live (real-time) queries, and leveraging all unique SurrealQL features for advanced database workflows. Use this skill whenever users need to write, adapt, or troubleshoot SurrealQL statements."
metadata:
  author: surrealdb
  version: "0.1.0"
---

# SurrealQL

A skill for writing and modifying SurrealQL queries to interact with SurrealDB databases.

SurrealQL is the official query language for SurrealDB. It is a modern, flexible, and powerful query language that is designed to be easy to learn and use.

## When to use this skill

Reference these guidelines when:

- Writing, modifying, or troubleshooting SurrealQL queries
- Designing or managing schemas
- Converting other query languages to SurrealQL

## Rules & Conventions

- SurrealQL is **NOT ANSI-SQL**. Never assume SQL knowledge from other databases applies. Always refer to the examples below or the documentation at https://surrealdb.com/docs for accurate syntax and behavior.
- When SurrealQL is stored in a file, it should have a `.surql` extension.
- SurrealQL is a relatively young language, and new features are being added regularly. Refer to the documentation at https://surrealdb.com/docs for the most up-to-date information.

## Statements

### Query Statements

| Statement | Purpose |
| --- | --- |
| [SELECT](https://surrealdb.com/docs/surrealql/statements/select) | Query records, traverse graphs, aggregate data |
| [CREATE](https://surrealdb.com/docs/surrealql/statements/create) | Create new records (errors if record exists) |
| [INSERT](https://surrealdb.com/docs/surrealql/statements/insert) | Insert one or more records or graph edges; supports `ON DUPLICATE KEY UPDATE` |
| [UPDATE](https://surrealdb.com/docs/surrealql/statements/update) | Update existing records (no-op if record doesn't exist) |
| [UPSERT](https://surrealdb.com/docs/surrealql/statements/upsert) | Insert a record, or update it if it already exists |
| [DELETE](https://surrealdb.com/docs/surrealql/statements/delete) | Delete records or graph edges |
| [RELATE](https://surrealdb.com/docs/surrealql/statements/relate) | Create graph edges between records |
| [LIVE SELECT](https://surrealdb.com/docs/surrealql/statements/live) | Stream real-time changes to a table |
| [KILL](https://surrealdb.com/docs/surrealql/statements/kill) | Cancel an active LIVE SELECT query |
| [LET](https://surrealdb.com/docs/surrealql/statements/let) | Assign a value to a parameter |
| [RETURN](https://surrealdb.com/docs/surrealql/statements/return) | Return a value from a block or function |

### Schema & Resource Statements

| Statement | Purpose |
| --- | --- |
| [DEFINE NAMESPACE](https://surrealdb.com/docs/surrealql/statements/define/namespace) | Define a namespace |
| [DEFINE DATABASE](https://surrealdb.com/docs/surrealql/statements/define/database) | Define a database |
| [DEFINE TABLE](https://surrealdb.com/docs/surrealql/statements/define/table) | Define a table (schemafull, schemaless, as view) |
| [DEFINE FIELD](https://surrealdb.com/docs/surrealql/statements/define/field) | Define a field with type, default, assertion |
| [DEFINE INDEX](https://surrealdb.com/docs/surrealql/statements/define/index) | Define an index (unique, search, vector) |
| [DEFINE EVENT](https://surrealdb.com/docs/surrealql/statements/define/event) | Define event triggers on a table |
| [DEFINE FUNCTION](https://surrealdb.com/docs/surrealql/statements/define/function) | Define a custom function |
| [DEFINE ANALYZER](https://surrealdb.com/docs/surrealql/statements/define/analyzer) | Define a search analyzer |
| [DEFINE ACCESS](https://surrealdb.com/docs/surrealql/statements/define/access) | Define authentication access methods (Bearer, JWT, Record) |
| [DEFINE API](https://surrealdb.com/docs/surrealql/statements/define/api) | Define an API endpoint |
| [DEFINE BUCKET](https://surrealdb.com/docs/surrealql/statements/define/bucket) | Define a storage bucket |
| [DEFINE CONFIG](https://surrealdb.com/docs/surrealql/statements/define/config) | Define a configuration |
| [DEFINE MODULE](https://surrealdb.com/docs/surrealql/statements/define/module) | Define a module |
| [DEFINE PARAM](https://surrealdb.com/docs/surrealql/statements/define/param) | Define a global parameter |
| [DEFINE SEQUENCE](https://surrealdb.com/docs/surrealql/statements/define/sequence) | Define an auto-incrementing sequence |
| [DEFINE USER](https://surrealdb.com/docs/surrealql/statements/define/user) | Define a system user |
| [ALTER](https://surrealdb.com/docs/surrealql/statements/alter) | Alter an existing resource definition |
| [REMOVE](https://surrealdb.com/docs/surrealql/statements/remove) | Remove any defined resource |
| [REBUILD](https://surrealdb.com/docs/surrealql/statements/rebuild) | Rebuild an index |
| [ACCESS](https://surrealdb.com/docs/surrealql/statements/access) | Manage access grants |
| [USE](https://surrealdb.com/docs/surrealql/statements/use) | Switch to a different namespace or database |
| [INFO](https://surrealdb.com/docs/surrealql/statements/info) | Inspect definitions for a resource |
| [SHOW](https://surrealdb.com/docs/surrealql/statements/show) | View changefeed for a table or database |

### Control Flow Statements

| Statement | Purpose |
| --- | --- |
| [BEGIN / COMMIT](https://surrealdb.com/docs/surrealql/statements/begin) | Begin and commit a manual transaction |
| [CANCEL](https://surrealdb.com/docs/surrealql/statements/cancel) | Cancel a transaction |
| [IF / ELSE](https://surrealdb.com/docs/surrealql/statements/if-else) | Conditional execution |
| [FOR](https://surrealdb.com/docs/surrealql/statements/for) | Iterate over values |
| [BREAK](https://surrealdb.com/docs/surrealql/statements/break) | Exit a FOR loop early |
| [CONTINUE](https://surrealdb.com/docs/surrealql/statements/continue) | Skip to next iteration in a FOR loop |
| [THROW](https://surrealdb.com/docs/surrealql/statements/throw) | Cancel execution and return an error |
| [SLEEP](https://surrealdb.com/docs/surrealql/statements/sleep) | Pause execution for a duration |

## References

For detailed querying patterns (filtering, graph traversal, aggregation,
subqueries), see [references/querying.md](references/querying.md).

For schema management patterns (tables, fields, indexes, events, access),
see [references/schema.md](references/schema.md).

For information about the values that can be stored in SurrealDB records,
see [references/values.md](references/values.md).

## Validation

Use the SurrealDB CLI to validate SurrealQL queries.

```bash
surreal validate query.surql
```

You can pass a glob pattern to validate multiple files at once.

## Formatting

When SurrealQL queries need to be well-formatted — for example when presenting
queries to users, generating migration files, or writing `.surql` files — use
the [`@surrealdb/surql-fmt`](https://www.npmjs.com/package/@surrealdb/surql-fmt)
CLI tool.

### Usage

```bash
# Format a file and print to stdout:
npx @surrealdb/surql-fmt query.surql

# Format files in-place:
npx @surrealdb/surql-fmt --write migrations/*.surql

# Check if files are already formatted (exits with code 1 if not):
npx @surrealdb/surql-fmt --check src/**/*.surql

# Format from stdin:
echo "SELECT * FROM person WHERE age>18" | npx @surrealdb/surql-fmt --stdin
```