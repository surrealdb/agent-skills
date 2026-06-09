---
title: Testing Framework
---

# Testing Framework

`surrealkit test` runs declarative TOML suites from `database/tests/suites/*.toml`.
By default each suite runs in an isolated ephemeral namespace/database and the
command exits non-zero (failing CI) if any case fails.

```bash
surrealkit test
surrealkit test --json-out database/tests/report.json   # machine-readable CI report
```

## CLI flags

| Flag | Behaviour |
| --- | --- |
| `--suite <glob>` | Only run matching suites |
| `--case <glob>` | Only run matching cases |
| `--tag <tag>` | Only run cases with this tag (repeatable) |
| `--fail-fast` | Stop on first failure |
| `--parallel <N>` | Run N suites concurrently (default `1`) |
| `--json-out <path>` | Write a machine-readable JSON report |
| `--no-setup` | Skip running `setup.surql` |
| `--no-sync` | Skip `sync` before tests |
| `--no-seed` | Skip seeding before tests |
| `--base-url <url>` | API base URL for `api_request` cases |
| `--timeout-ms <ms>` | Per-case timeout |
| `--keep-db` | Keep the ephemeral test database for inspection |

## Global config

Global test settings live in `database/tests/config.toml`:

```toml
[defaults]
timeout_ms = 10000
base_url = "http://localhost:8000"

[actors.root]
kind = "root"
```

Env fallbacks: `SURREALKIT_TEST_BASE_URL`, `SURREALKIT_TEST_TIMEOUT_MS`, and
`SURREALDB_HOST` / `DATABASE_HOST` (used as the API base URL fallback).

## Suite shape

```toml
name = "security_smoke"
tags = ["smoke", "security"]

[[cases]]
name = "guest_cannot_create_order"
kind = "sql_expect"
actor = "guest"
sql = "CREATE order CONTENT { total: 10 };"
allow = false
error_contains = "permission"

[[cases]]
name = "orders_api_returns_200"
kind = "api_request"
actor = "root"
method = "GET"
path = "/api/orders"
expected_status = 200

[[cases.body_assertions]]
path = "0.id"
exists = true
```

## Case kinds

| `kind` | Tests |
| --- | --- |
| `sql_expect` | A SQL statement is allowed/denied and returns expected values |
| `permissions_matrix` | A grid of record-level permission rules for an actor |
| `schema_metadata` | Schema metadata (e.g. `INFO FOR TABLE`) contains expected content |
| `schema_behavior` | Schema behavioural assertions |
| `api_request` | An HTTP API endpoint returns expected status/body |

### `sql_expect`

Use `allow` (`true`/`false`) and optionally `error_contains` for denied cases.
Add `[[cases.assertions]]` to check returned values. To compare a field against
the authenticated actor, use `equals_auth` with `$auth` or `$auth.<property>`:

```toml
[[cases]]
name = "user_can_create_calendar"
kind = "sql_expect"
actor = "user_alice"
sql = "CREATE calendar CONTENT { name: 'Alice Personal' };"
allow = true

[[cases.assertions]]
path = "0.owner"
equals_auth = "$auth.id"
```

### `permissions_matrix`

```toml
[[cases]]
name = "reader_permissions"
kind = "permissions_matrix"
actor = "reader"
table = "order"
record_id = "perm_test"

[[cases.rules]]
action = "select"
allow = true

[[cases.rules]]
action = "update"
allow = false
error_contains = "permission"
```

### `api_request`

Set `method`, `path`, `expected_status`, and optional `[[cases.body_assertions]]`
(each with a `path` and a check such as `exists` or an expected value).

## Actors

Actors define who runs each case. Configure them in `config.toml` (`[actors.*]`)
or per suite. Secrets come from environment variables via `*_env` keys.

```toml
[actors.reader]
kind = "database"
namespace = "app"
database = "main"
username_env = "TEST_DB_READER_USER"
password_env = "TEST_DB_READER_PASS"

[actors.access_user]
kind = "record"
access = "app_access"
signup_params = { email = "viewer@example.com", password = "viewer-password" }
signin_params = { email = "viewer@example.com", password = "viewer-password" }

[actors.jwt_actor]
kind = "token"
token_env = "TEST_API_JWT"

[actors.custom_client]
kind = "headers"
headers = { "x-tenant-id" = "tenant_a" }
```

| `kind` | Authenticates as |
| --- | --- |
| `root` | Root user |
| `database` | A namespace/database user (`username_env` / `password_env`) |
| `record` | A record-access user (`access`, `signup_params`, `signin_params`) |
| `token` | A pre-issued JWT (`token_env`) |
| `headers` | Custom HTTP headers (for `api_request` cases) |

For `record` actors, `signup_params` is optional and runs before
authentication; `signin_params` is used for the signin step (legacy `params`
still works as a signin alias).

## CI

```bash
surrealkit test --json-out database/tests/report.json
```

Exits non-zero if any case fails, and the JSON report gives machine-readable
results for CI pipelines. The Docker image (`ghcr.io/surrealdb/surrealkit`)
exits on completion, which suits "apply schema then run tests" pipelines in
Docker Compose alongside SurrealDB.
