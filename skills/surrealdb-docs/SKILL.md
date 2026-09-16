---
name: surrealdb-docs
description: "Retrieve and display SurrealDB documentation via SSH."
---

# SurrealDB SSH Setup

Choose how to add SurrealDB docs to your workflow:

## Option 1: Append to AGENTS.md

```bash
ssh surrealdb.sh agents >> AGENTS.md
```

## Option 2: Let your agent self-configure

```bash
ssh surrealdb.sh setup | claude
```

## Option 3: Interactive exploration

```bash
ssh surrealdb.sh
```

Then use standard bash commands to browse docs at `/surrealdb/docs`.
