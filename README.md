# me-skills

English | [简体中文](README.zh-CN.md)

A distribution and coordination repository for the `db-query` and `loki-query`
Agent Skills and their CLIs. Each independent subproject owns its CLI and the
authoritative skill source; this repository publishes synchronized copies under
`skills/` for installation and discovery.

## Sources of truth

| Skill | Purpose | Published copy | GitHub project |
| --- | --- | --- | --- |
| `db-query` | Runs guarded, read-only MySQL queries through an explicitly selected profile. It requires database-qualified table names, bounds query cost, and requires approval of the exact profile and SQL before production access. | [`skills/db-query`](skills/db-query) | [Nza6920/db-cli](https://github.com/Nza6920/db-cli) |
| `loki-query` | Queries Grafana Loki through an explicitly selected profile. It bounds query windows and attempts, then separates log evidence, inferences, and unresolved checks. | [`skills/loki-query`](skills/loki-query) | [Nza6920/grafana-loki-query-cli](https://github.com/Nza6920/grafana-loki-query-cli) |

In this workspace, the canonical entry points are:

```text
db-cli/.agents/skills/db-query/SKILL.md
loki-query/.agents/skills/loki-query/SKILL.md
```

The `db-cli/` and `loki-query/` directories are ignored, independent Git
worktrees. Make skill changes in the corresponding subproject first, then
synchronize the complete skill directory into `skills/`. Commit the source
change in the subproject and the published copy in this repository.

## Usage

1. Install and configure the CLI from its GitHub project linked above.
2. Copy or link `skills/<name>` into a skills directory discoverable by your
   Agent.
3. Ensure the CLI is on `PATH`, then invoke the skill explicitly:

```text
$db-query use the uat profile to query the latest 20 rows from logistics.t_waybill
$loki-query use the prod profile to query error logs for order 252143 from the last 30 minutes
```

Both skills require explicit invocation so ordinary conversations cannot access
database or log environments. Connection details and tokens remain in each
CLI's profiles and environment variables rather than this repository.

## Workspace layout

```text
skills/                         # synchronized published copies
├── db-query/
│   ├── SKILL.md
│   └── agents/openai.yaml
└── loki-query/
    ├── SKILL.md
    └── agents/openai.yaml
db-cli/                         # independent Git worktree
└── .agents/skills/db-query/    # db-query source of truth
loki-query/                     # independent Git worktree
└── .agents/skills/loki-query/  # loki-query source of truth
```
