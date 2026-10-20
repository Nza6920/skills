# me-skills

English | [简体中文](README.zh-CN.md)

A distribution and coordination repository for the `db-query`, `loki-query`,
and `redis-query` Agent Skills. The database and Loki subprojects own their CLIs
and authoritative skill sources; this repository publishes synchronized copies
under `skills/`. The Redis skill is maintained directly here and uses the
official `redis-cli`.

## Sources of truth

| Skill | Purpose | Skill directory | CLI project or documentation |
| --- | --- | --- | --- |
| `db-query` | Runs guarded, read-only MySQL queries through an explicitly selected profile. It requires database-qualified table names, bounds query cost, and requires approval of the exact profile and SQL before production access. | [`skills/db-query`](skills/db-query) | [Nza6920/db-cli](https://github.com/Nza6920/db-cli) |
| `loki-query` | Queries Grafana Loki through an explicitly selected profile. It bounds query windows and attempts, then separates log evidence, inferences, and unresolved checks. | [`skills/loki-query`](skills/loki-query) | [Nza6920/grafana-loki-query-cli](https://github.com/Nza6920/grafana-loki-query-cli) |
| `redis-query` | Runs bounded, read-only Redis queries with an explicitly selected local environment. It requires approval of the exact target and command for production access. | [`skills/redis-query`](skills/redis-query)  | [Official Redis CLI](https://redis.io/docs/latest/develop/tools/cli/) |

In this workspace, the canonical entry points are:

```text
db-cli/.agents/skills/db-query/SKILL.md
loki-query/.agents/skills/loki-query/SKILL.md
skills/redis-query/SKILL.md
```

The `db-cli/` and `loki-query/` directories are ignored, independent Git
worktrees. Make skill changes in the corresponding subproject first, then
synchronize the complete skill directory into `skills/`. Commit the source
change in the subproject and the published copy in this repository. Edit
`skills/redis-query/` directly; it has no independent CLI subproject.

## Usage

1. Install and configure the corresponding CLI using the project or official
   documentation linked above. For Redis, create one local
   `~/.redis-env.<environment>` file per environment, following the
   [configuration template](skills/redis-query/references/connection.md).
2. Copy or link `skills/<name>` into a skills directory discoverable by your
   Agent.
3. Ensure the CLI is on `PATH`, then invoke the skill explicitly:

```text
$db-query use the uat profile to query the latest 20 rows from logistics.t_waybill
$loki-query use the prod profile to query error logs for order 252143 from the last 30 minutes
$redis-query use the uat environment to inspect the type, TTL, and a bounded preview of cache:order:252143
```

All three skills require explicit invocation so ordinary conversations cannot
access database or log environments. Connection details and credentials remain
in local configuration and environment variables rather than this repository.

Redis environment files use `REDISCLI_AUTH`, `REDIS_HOST`, and `REDIS_PORT`,
plus the required `REDIS_ENV_TYPE=production|non-production`. Authentication
uses only the complete password in `REDISCLI_AUTH`, preserving characters such
as `@` without splitting or URL-encoding them.
Optional settings cover DB (default 0), TLS, and Cluster. Missing configuration
prompts local setup before any connection; passwords stay out of the
conversation. Default reads use at most ten scan calls with `COUNT 100`, display
up to 100 items, and preview String values up to 4 KiB. Scan counts are work
hints, not hard response limits.

## Workspace layout

```text
skills/                         # published skills
├── db-query/
│   ├── SKILL.md
│   └── agents/openai.yaml
├── loki-query/
│   ├── SKILL.md
│   └── agents/openai.yaml
└── redis-query/                 # root-owned source of truth
    ├── SKILL.md
    ├── agents/openai.yaml
    └── references/
        ├── connection.md
        └── queries.md
db-cli/                         # independent Git worktree
└── .agents/skills/db-query/    # db-query source of truth
loki-query/                     # independent Git worktree
└── .agents/skills/loki-query/  # loki-query source of truth
```
