# me-skills

English | [简体中文](README.zh-CN.md)

A personal collection of Agent Skills that provides repeatable, guarded workflows for database queries and log investigations.

Available skills live under [`skills/`](skills/). Each skill guides an Agent in using its corresponding CLI; the CLI projects are developed and published independently of this repository.

## Available Skills

| Skill | Purpose | CLI | CLI source |
| --- | --- | --- | --- |
| [`db-query`](skills/db-query/SKILL.md) | Runs guarded, read-only MySQL queries through an explicitly selected profile. It requires database-qualified table names, bounds detail queries, and presents the SQL for explicit approval before accessing production. | `db-query` | [Nza6920/db-cli](https://github.com/Nza6920/db-cli) |
| [`loki-query`](skills/loki-query/SKILL.md) | Queries Grafana Loki logs through an explicitly selected profile. It bounds query windows and refinement attempts, then reports log evidence, inferences, and unresolved checks separately. | `loki-query` | [Nza6920/grafana-loki-query-cli](https://github.com/Nza6920/grafana-loki-query-cli) |

Both skills require explicit invocation, preventing ordinary conversations from accidentally accessing databases or log systems:

```text
$db-query use the uat profile to query the latest 20 rows from logistics.t_waybill
$loki-query use the prod profile to query error logs for order 252143 from the last 30 minutes
```

## Usage

1. Install and configure the CLI from its GitHub repository linked above.
2. Copy or link the required `skills/<name>` directory into a skills directory discoverable by your Agent.
3. Ensure the CLI is available on `PATH`, then invoke the skill explicitly with `$<skill-name>`.

Database connection details and Grafana tokens are managed through each CLI's profiles and environment variables. They should not be stored in this repository.

## Directory Structure

```text
skills/
├── db-query/
│   └── SKILL.md
└── loki-query/
    └── SKILL.md
```
