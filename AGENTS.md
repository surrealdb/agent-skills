# AGENTS.md

Guidance for AI coding agents working with this repository.

## Repository Structure

```
skills/
  {skill-name}/
    SKILL.md
    references/ (optional)
```

## Creating a New Skill

Skills follow the [Agent Skills Open Standard](https://agentskills.io/).

1. Create directory: `mkdir -p skills/{skill-name}/references`
2. Create `SKILL.md` with YAML frontmatter (`name`, `description`) and
   Markdown body
3. Add reference files under `references/` if the skill body exceeds ~500 lines
4. Ensure the `name` in frontmatter matches the directory name
5. Update `README.md` to describe the skill and when to use it

### Frontmatter (Required)

```yaml
---
name: skill-name
description: What this skill does and when to use it.
metadata:
  author: surrealdb
  version: "0.1.0"
---
```

| Field         | Required | Constraints                                                     |
| ------------- | -------- | --------------------------------------------------------------- |
| `name`        | Yes      | 1-64 chars. Lowercase alphanumeric and hyphens only.            |
| `description` | Yes      | 1-1024 chars. Describe what the skill does AND when to use it.  |
| `metadata`    | No       | Arbitrary key-value pairs (e.g., `author`, `version`).          |

### Body Guidelines

- Keep under 500 lines; move detailed content to `references/`
- Use imperative form ("Create the table", not "You should create the table")
- Include concise examples over verbose explanations
- Version bumps: any change to `SKILL.md` or `references/` should bump `version`

## What NOT to Include in Skills

Skills should only contain essential files. Do NOT create README, CHANGELOG,
or installation guides inside skill directories.
