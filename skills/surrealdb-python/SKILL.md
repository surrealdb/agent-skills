---
name: surrealdb-python
description: "Using SurrealDB with the Python SDK, covering both client/server mode (WebSocket) and embedded mode (in-memory or file-based). Use when connecting to SurrealDB from Python, using the surrealdb Python package, running SurrealDB embedded without a server, or performing CRUD operations from Python code. Triggers: surrealdb Python, Surreal(), AsyncSurreal(), Python SDK, embedded SurrealDB, mem://, file://."
metadata:
  author: surrealdb
  version: "0.1.1"
---

# SurrealDB Python SDK

## Running SurrealDB (Server Mode)

Persist data with RocksDB:

```bash
surreal start -u root -p root rocksdb:database
```

In-memory:

```bash
surreal start -u root -p root
```

## Client/Server Mode

Connect via WebSocket using the `Surreal` context manager:

```python
from surrealdb import Surreal

with Surreal("ws://localhost:8000/rpc") as db:
    db.signin({"username": "root", "password": "root"})
    db.use("namespace_test", "database_test")

    db.create(
        "person",
        {
            "user": "me",
            "password": "safe",
            "marketing": True,
            "tags": ["python", "documentation"],
        },
    )

    print(db.select("person").execute())

    print(db.update("person", {
        "user": "you",
        "password": "very_safe",
        "marketing": False,
        "tags": ["Awesome"],
    }))

    print(db.delete("person").execute())

    db.query("""
    insert into person {
        user: 'me',
        password: 'very_safe',
        tags: ['python', 'documentation']
    };
    """).execute()

    print(db.query("select * from person").execute())

    print(db.query("""
    update person content {
        user: 'you',
        password: 'more_safe',
        tags: ['awesome']
    };
    """).execute())

    print(db.query("delete person").execute())
```

### Builders and `.execute()`

`query()`, `select()`, `delete()` and the no-data forms of `create()` /
`insert()` return a builder. With the sync client, **nothing is sent until
you call `.execute()`** (or iterate the builder); a dropped builder is a silent
no-op and emits a `RuntimeWarning`. `create()` / `update()` with a data
argument run immediately. With `AsyncSurreal`, `await` the call instead
(`await db.query(...)`), and `.execute()` also works.

`query()` returns a list with one result per statement, even for a single
statement. `.first()` runs the query and returns just the first statement's
result:

```python
people = db.query("select * from person").first()
```

## Embedded Mode

Run SurrealDB directly inside your Python process — no server required.
Install with `pip install "surrealdb[embedded]"`.

- **In-memory**: `Surreal("mem://")` / `AsyncSurreal("mem://")`
- **File-based persistence**: `Surreal(f"file://{db_path}")` / `AsyncSurreal(f"file://{db_path}")`

See [references/embedded.md](references/embedded.md) for a complete async
example with file-based persistence.
