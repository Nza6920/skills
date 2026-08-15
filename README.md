# me-skills

English | [简体中文](README.zh-CN.md)

A coordination index for the `db-query` and `loki-query` Agent Skills and their
CLIs. Each independent subproject owns both its CLI implementation and its
repository-local skill; this root repository does not keep duplicate skill
copies.

## Sources of truth

| Skill | Purpose | Authoritative source |
| --- | --- | --- |
| `db-query` | Runs guarded, read-only MySQL queries through an explicitly selected profile. It requires database-qualified table names, bounds query cost, and requires approval of the exact profile and SQL before production access. | [Nza6920/db-cli `.agents/skills/db-query`](https://github.com/Nza6920/db-cli/tree/main/.agents/skills/db-query) |
| `loki-query` | Queries Grafana Loki through an explicitly selected profile. It bounds query windows and attempts, then separates log evidence, inferences, and unresolved checks. | [Nza6920/grafana-loki-query-cli `.agents/skills/loki-query`](https://github.com/Nza6920/grafana-loki-query-cli/tree/master/.agents/skills/loki-query) |

In this workspace, the canonical entry points are:

```text
db-cli/.agents/skills/db-query/SKILL.md
loki-query/.agents/skills/loki-query/SKILL.md
```

The `db-cli/` and `loki-query/` directories are ignored, independent Git
worktrees. Make and commit skill changes in the corresponding subproject. A
root-level `skills/` directory is intentionally absent.

## Usage

1. Install and configure the CLI from its project linked above.
2. Copy or link the project's `.agents/skills/<name>` directory into a skills
   directory discoverable by your Agent.
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
db-cli/                         # independent Git worktree
└── .agents/skills/db-query/    # db-query source of truth
loki-query/                     # independent Git worktree
└── .agents/skills/loki-query/  # loki-query source of truth
```
