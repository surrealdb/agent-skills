---
name: surrealdb-docs
description: "Retrieve and display SurrealDB documentation via SSH."
---

# SurrealDB Documentation — Agent Access

> Browse SurrealDB docs directly in your terminal via SSH.

## Quick start

```bash
ssh surrealdb.sh grep -rl 'SELECT' /surrealdb/docs
ssh surrealdb.sh cat /surrealdb/docs/surrealql/statements/select.mdx
ssh surrealdb.sh find /surrealdb/docs -name '*.mdx' | head -20
```

## Available commands

All standard Unix text utilities work inside the sandbox:

- `grep` — search across documentation topics
- `find` — locate relevant pages
- `cat` — read full documentation files
- `head` / `tail` — skim content without consuming excessive context
- `ls` — list directory contents
- `wc` — count lines/words

## Tips for agents

1. Start with `find /surrealdb/docs -name '*.mdx'` to discover available pages
2. Use `grep -rl '<keyword>' /surrealdb/docs` to find relevant files
3. Use `head -50` to skim before reading full files
4. The docs mirror surrealdb.com/docs — same markdown source
