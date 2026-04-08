---
name: surrealql
description: "SurrealQL query language reference for SurrealDB. Use when writing SurrealQL queries (SELECT, CREATE, UPDATE, DELETE, RELATE), designing schemas (DEFINE TABLE, DEFINE FIELD, DEFINE INDEX), working with graph relationships and record IDs, migrating from traditional SQL, or setting up live queries. Triggers: SurrealDB, SurrealQL, surql, record IDs, graph queries, RELATE, LIVE SELECT."
metadata:
  author: surrealdb
  version: "0.1.0"
---

# SurrealQL

**IMPORTANT**: SurrealQL is NOT ANSI-SQL. Never assume SQL knowledge from other databases applies. Always refer to the examples below or the documentation at https://surrealdb.com/docs for accurate syntax and behavior.

## Key Concepts

### Record IDs

SurrealDB uses `table:id` for record identifiers:

- `person:john` — record with ID "john" in "person" table
- `article:surreal_intro` — record with ID "surreal_intro" in "article" table

Each record has a unique `id` field which cannot be changed once specified.

### Key Differences from SQL

- Every row is called a **Record** (similar to a MongoDB document, supports nested objects and arrays)
- Record IDs use `table:id` format
- Graph traversal uses arrow syntax:
  - `->` outbound connections
  - `<-` inbound connections
  - `<->` any direction
- Use `RELATE` for graph relationships instead of foreign keys
- Use `MERGE` / `PATCH` on `UPDATE` to preserve existing data

### Update Modes

- **CONTENT** (replace): replaces entire record content
- **MERGE**: merges new data with existing data
- **PATCH**: applies JSON patch operations

## Best Practices

1. Use specific record IDs when known for better performance
2. Use parameterized queries to prevent injection when dealing with user input
3. Use `type::table($table)` to safely parameterize table names
4. Use `MERGE` / `PATCH` modes when updating to preserve existing data
5. Use `RELATE` to model graph data instead of foreign keys
6. Define schemas for data validation and consistency
7. Create indexes on frequently queried fields
8. Use `LIVE` queries for real-time updates when needed

## SELECT

```surql
SELECT * FROM type::table($table);

SELECT name, age FROM type::table($table);

SELECT * FROM type::table($table):john;

SELECT * FROM type::table($table) WHERE age > 25;

SELECT * FROM type::table($table) WHERE age > 25 ORDER BY name LIMIT 10;

SELECT * FROM type::table($table) SPLIT ON age, city;

SELECT count(), age FROM type::table($table) GROUP BY age;

SELECT * FROM type::table($table) ORDER BY name LIMIT 10 START AT 20;

-- Graph queries
SELECT * FROM type::table($table) WHERE ->knows->person.age > 30;

SELECT * FROM type::table($table)
  WHERE ->wrote->article->has->category.name = 'Technology';

-- Aggregation
SELECT count() FROM type::table($table) GROUP BY author;

-- Subqueries
SELECT * FROM type::table($table) WHERE id IN (SELECT author FROM article);

-- Complex query with multiple clauses
SELECT * FROM type::table($table)
  WHERE age > 25 AND city = 'NYC'
  SPLIT ON age
  GROUP BY age, city
  ORDER BY age DESC
  LIMIT 10
  START AT 5;
```

## CREATE

```surql
CREATE person:john CONTENT {
    name: 'John Doe',
    age: 30,
    email: 'john@example.com'
};

CREATE person CONTENT [
    { name: 'Alice', age: 25 },
    { name: 'Bob', age: 35 }
];

CREATE person:alice CONTENT { name: 'Alice', age: 25 };
```

## UPDATE

```surql
-- Replace entire record
UPDATE person:john CONTENT {
    name: 'John Doe',
    age: 31,
    city: 'New York'
};

-- Merge with existing data
UPDATE person:john MERGE {
    age: 31,
    city: 'New York'
};

-- JSON patch
UPDATE person:john PATCH [
    { op: 'replace', path: '/age', value: 31 },
    { op: 'add', path: '/city', value: 'New York' }
];

-- Bulk update
UPDATE person SET age += 1 WHERE age < 30;
```

## DELETE

```surql
DELETE person:john;

DELETE person;

DELETE person WHERE age < 18;
```

## RELATE

```surql
RELATE person:john->knows->person:jane;

RELATE person:john->wrote->article:surreal_guide CONTENT {
    date: '2024-01-15',
    word_count: 1500
};
```

## Schema Definition

```surql
DEFINE TABLE person SCHEMAFULL;

DEFINE FIELD name ON TABLE person TYPE string;
DEFINE FIELD age ON TABLE person TYPE int;
DEFINE FIELD email ON TABLE person TYPE string;

DEFINE INDEX idx_name ON TABLE person COLUMNS name;
DEFINE INDEX idx_age ON TABLE person COLUMNS age;
```

## Live Queries

```surql
LIVE SELECT * FROM person;

LIVE SELECT * FROM person WHERE age > 25;

LIVE SELECT * FROM person WHERE ->knows->person.age > 30;
```

## Advanced Features

### Nested Object and Array Access

```surql
SELECT name, profile.city FROM person;

SELECT * FROM person WHERE tags CONTAINS 'developer';

SELECT * FROM person WHERE profile.age > 25;
```

### Time and Date Operations

```surql
SELECT * FROM article WHERE created_at > '2024-01-01';

SELECT * FROM event WHERE time::day(created_at) = time::day(now());
```

### String Operations

```surql
SELECT * FROM person WHERE name CONTAINS 'John';

SELECT * FROM person WHERE string::uppercase(name) = 'JOHN';
```

## Common Pattern: Blog System

```surql
CREATE person:john CONTENT { name: 'John Doe', email: 'john@example.com' };
CREATE article:surreal_guide CONTENT { title: 'SurrealDB Guide', content: '...' };

RELATE person:john->wrote->article:surreal_guide;
RELATE article:surreal_guide->has->category:technology;

-- Articles by author
SELECT * FROM article WHERE ->wrote->person.name = 'John Doe';

-- Authors by article category
SELECT * FROM person WHERE ->wrote->article->has->category.name = 'Technology';
```
