# SurrealDB Agent Skills

Official [SurrealDB](https://surrealdb.com) agent skills for use in agentic
workflows. Agent Skills are folders of instructions, scripts, and resources
that agents can discover and use to do things more accurately and efficiently.
Compatible with most AI agents including Claude Code, GitHub Copilot, Cursor,
Cline, and many others.

The skills in this repo follow the [Agent Skills](https://agentskills.io/)
format.

## Installation

### Install all skills

```bash
npx skills add surrealdb/agent-skills
```

### Install a specific skill

```bash
npx skills add surrealdb/agent-skills --skill surrealql
npx skills add surrealdb/agent-skills --skill surrealdb-vector
npx skills add surrealdb/agent-skills --skill surrealdb-python
```

### Local install from repository

1. Clone the repository:

   ```bash
   git clone https://github.com/surrealdb/agent-skills.git
   ```

2. Copy the `skills/` directory to the location where your coding agent reads
   its skills or context files. Refer to your agent's documentation for the
   correct path.

## Available Skills

<details>
<summary><strong>surrealql</strong></summary>

Core SurrealQL query language reference covering syntax, best practices, schema
definitions, graph relationships, and common patterns.

**Use when:**

- Writing SurrealQL queries (SELECT, CREATE, UPDATE, DELETE, RELATE)
- Designing schemas with DEFINE TABLE, DEFINE FIELD, and DEFINE INDEX
- Working with graph relationships and record IDs
- Migrating from traditional SQL to SurrealQL
- Setting up live queries for real-time updates

</details>

<details>
<summary><strong>surrealdb-vector</strong></summary>

Vector search with SurrealDB using HNSW indexes, KNN queries, and similarity
scoring.

**Use when:**

- Creating HNSW vector indexes on tables
- Querying vectors with KNN distance operators
- Building semantic search, RAG pipelines, or recommendation systems
- Tuning HNSW parameters (EFC, M, M0, distance function, type)

</details>

<details>
<summary><strong>surrealdb-python</strong></summary>

Using SurrealDB with the Python SDK, covering both client/server mode
(WebSocket) and embedded mode (in-memory or file-based persistence).

**Use when:**

- Connecting to SurrealDB from Python applications
- Using the `surrealdb` Python package (sync or async)
- Running SurrealDB embedded in Python without a server
- Performing CRUD operations from Python code

</details>

## Usage

Skills are automatically available once installed. The agent will use them when
relevant tasks are detected.

**Examples:**

```
Write a SurrealQL query to find all users who follow each other
```

```
Create an HNSW vector index for semantic search on my documents table
```

```
Connect to SurrealDB from my Python application
```

## Skill Structure

Each skill follows the [Agent Skills Open Standard](https://agentskills.io/):

- `SKILL.md` — Required skill manifest with frontmatter (name, description,
  metadata)
- `references/` — (Optional) Reference files for detailed documentation

## Resources

- [SurrealDB Documentation](https://surrealdb.com/docs)
- [SurrealDB Agent Rules](https://surrealdb.com/docs/integrations/agent-rules)
- [Agent Skills Specification](https://agentskills.io/specification)
