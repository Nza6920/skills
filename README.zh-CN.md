# me-skills

[English](README.md) | 简体中文

本仓库是 `db-query`、`loki-query` Agent Skills 及其 CLI 的协调索引。每个独立
子项目同时维护 CLI 实现和仓库内 skill；根仓库不再保留重复的 skill 副本。

## 唯一事实来源

| Skill | 作用 | 权威来源 |
| --- | --- | --- |
| `db-query` | 通过明确选择的 profile 执行受保护的 MySQL 只读查询。要求使用数据库限定表名、限制查询成本，并在访问 production 前批准完整且未变化的 profile 与 SQL。 | [Nza6920/db-cli `.agents/skills/db-query`](https://github.com/Nza6920/db-cli/tree/main/.agents/skills/db-query) |
| `loki-query` | 通过明确选择的 profile 查询 Grafana Loki。限制查询时间窗和次数，并将日志证据、推断与待核实事项分开报告。 | [Nza6920/grafana-loki-query-cli `.agents/skills/loki-query`](https://github.com/Nza6920/grafana-loki-query-cli/tree/master/.agents/skills/loki-query) |

在当前工作区中，权威入口为：

```text
db-cli/.agents/skills/db-query/SKILL.md
loki-query/.agents/skills/loki-query/SKILL.md
```

`db-cli/` 和 `loki-query/` 是被根仓库忽略的独立 Git worktree。Skill 变更应在
对应子项目中修改并提交。根目录有意不再设置 `skills/` 目录。

## 使用方式

1. 从上表对应项目安装并配置 CLI。
2. 将项目中的 `.agents/skills/<name>` 目录复制或链接到 Agent 可发现的
   skills 目录。
3. 确认 CLI 已加入 `PATH`，再显式调用 skill：

```text
$db-query 使用 uat profile 查询 logistics.t_waybill 中最近 20 条记录
$loki-query 使用 prod profile 查询最近 30 分钟内订单 252143 的异常日志
```

两个 skill 都必须显式调用，避免普通对话访问数据库或日志环境。数据库连接
信息和 Grafana Token 保留在各 CLI 的 profile 与环境变量中，不写入本仓库。

## 工作区结构

```text
db-cli/                         # 独立 Git worktree
└── .agents/skills/db-query/    # db-query 唯一事实来源
loki-query/                     # 独立 Git worktree
└── .agents/skills/loki-query/  # loki-query 唯一事实来源
```
