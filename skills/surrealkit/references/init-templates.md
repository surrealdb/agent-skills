---
title: Project Init & Templates
---

# Project Init & Templates

`surrealkit init` scaffolds a project from a template and lets you choose which
optional features to include. It always writes the base layout first
(`schema/`, `rollouts/`, `snapshots/`, `seed/`, `tests/`, `setup.surql`,
`surrealkit.toml`), then adds the files contributed by the features you pick.

## Interactive

```bash
surrealkit init
```

In a terminal this shows a checklist of the template's features. Pick the ones
you want and SurrealKit writes their schema, seed, and test files into
`database/`.

## Non-interactive

When there is no terminal (e.g. CI) or you pass any of these flags, init runs
without prompting:

| Flag | Behaviour |
| --- | --- |
| `--feature <id>` | Enable a feature by id. Repeatable; pulls in what it requires |
| `-y`, `--yes` | Take the template's default features |
| `--minimal` | Scaffold the base project only, with no features |
| `--force` | Overwrite files that already exist (default is to skip them) |
| `--template <name>` | Use a named bundled template (default: `default`) |
| `--from <src>` | Use an external template (overrides `--template`) |

```bash
surrealkit init --feature organizations --feature teams
surrealkit init -y
surrealkit init --minimal
```

Selecting a feature also adds any features it requires, and init prints what it
added.

## Using your own template

Point `--from` at a local path or a git repository:

```bash
surrealkit init --from ./path/to/template
surrealkit init --from https://github.com/your-org/your-template.git
surrealkit init --from https://github.com/your-org/your-template.git#v1.0.0
surrealkit init --from https://github.com/your-org/your-template.git#v1.0.0:subdir
```

Git sources are cloned with `git clone --depth 1`, so `git` must be on your
PATH. Pin a branch, tag, or commit with `#rev`, and target a subdirectory with
`#rev:subdir`.

## Template layout

A template is a directory with a `template.toml` manifest plus the files each
feature contributes:

```toml
schema_version = 1
name = "default"
display_name = "My starter"
description = "Shown above the feature checklist"

[[features]]
id = "organizations"
name = "Organizations"
description = "Shown next to the feature in the checklist"
default = false
schema   = ["schema/organization/organization.surql"]
seed     = ["seed/organization_permissions.surql"]
suites   = ["tests/suites/organization.toml"]
fixtures = ["tests/fixtures/organization_seed.surql"]

[[features]]
id = "teams"
name = "Teams"
requires = ["organizations"]
schema = ["schema/team/team.surql"]
```

Each feature lists the files it adds, grouped by where they land:

- `schema` files → `database/schema/`
- `seed` files → `database/seed/`
- `suites` files → `database/tests/suites/`
- `fixtures` files → `database/tests/fixtures/`

Set `default = true` to pre-check a feature in the prompt and include it with
`-y`. Use `requires` to declare dependencies on other features.

## Bundled template

The bundled `default` template provides an organization and access-control
model with four opt-in features:

- **Organizations** — organizations, roles that bundle permissions, a per-app
  permission catalog, employees, and invitations.
- **Teams** — teams within an organization, with per-member roles.
- **Organization units** — a department and region hierarchy with unit-scoped
  permissions.
- **Subsidiaries and delegation** — parent and child organizations with
  cross-org delegated permissions.

Teams, units, and subsidiaries each require the organizations feature.
