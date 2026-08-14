# me-skills

[English](README.md) | 简体中文

个人维护的 Agent Skills 集合，主要用于为数据库查询和日志排查提供可重复、带安全边界的操作流程。

可用 skill 位于 [`skills/`](skills/) 目录。每个 skill 负责指导 Agent 如何调用对应的 CLI；CLI 本身是独立项目，不随本仓库发布。

## 可用 Skills

| Skill | 作用 | 对应 CLI | CLI 源码 |
| --- | --- | --- | --- |
| [`db-query`](skills/db-query/SKILL.md) | 通过指定 profile 执行受保护的 MySQL 只读查询。要求使用数据库限定表名、限制明细查询行数，并在访问生产环境前展示 SQL、等待明确确认。 | `db-query` | [Nza6920/db-cli](https://github.com/Nza6920/db-cli) |
| [`loki-query`](skills/loki-query/SKILL.md) | 通过指定 profile 查询 Grafana Loki 日志。限制单次查询时间窗和迭代次数，并将日志证据、推断与待确认项分开报告。 | `loki-query` | [Nza6920/grafana-loki-query-cli](https://github.com/Nza6920/grafana-loki-query-cli) |

这两个 skill 均采用显式调用，避免在普通对话中意外访问数据库或日志系统：

```text
$db-query 使用 uat profile 查询 logistics.t_waybill 中最近 20 条记录
$loki-query 使用 prod profile 查询最近 30 分钟内订单 252143 的异常日志
```

## 使用方式

1. 从上表对应的 GitHub 仓库安装并配置 CLI。
2. 将所需的 `skills/<name>` 目录复制或链接到 Agent 可发现的 skills 目录。
3. 确认 CLI 已加入 `PATH`，再通过 `$<skill-name>` 显式调用 skill。

数据库连接信息和 Grafana Token 由各 CLI 的 profile 与环境变量管理，不应写入本仓库。

## 目录结构

```text
skills/
├── db-query/
│   └── SKILL.md
└── loki-query/
    └── SKILL.md
```
