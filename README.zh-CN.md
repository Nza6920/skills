# me-skills

[English](README.md) | 简体中文

本仓库用于分发和协调 `db-query`、`loki-query` Agent Skills 及其 CLI。每个独立
子项目维护 CLI 和权威 Skill 源文件；本仓库在 `skills/` 下发布同步副本，供安装
和发现。

## 唯一事实来源

| Skill | 作用 | 发布副本 | GitHub 项目 |
| --- | --- | --- | --- |
| `db-query` | 通过明确选择的 profile 执行受保护的 MySQL 只读查询。要求使用数据库限定表名、限制查询成本，并在访问 production 前批准完整且未变化的 profile 与 SQL。 | [`skills/db-query`](skills/db-query) | [Nza6920/db-cli](https://github.com/Nza6920/db-cli) |
| `loki-query` | 通过明确选择的 profile 查询 Grafana Loki。限制查询时间窗和次数，并将日志证据、推断与待核实事项分开报告。 | [`skills/loki-query`](skills/loki-query) | [Nza6920/grafana-loki-query-cli](https://github.com/Nza6920/grafana-loki-query-cli) |

在当前工作区中，权威入口为：

```text
db-cli/.agents/skills/db-query/SKILL.md
loki-query/.agents/skills/loki-query/SKILL.md
```

`db-cli/` 和 `loki-query/` 是被根仓库忽略的独立 Git worktree。应先在对应
子项目中修改 Skill，再将完整 Skill 目录同步到 `skills/`。源文件变更在子项目
中提交，发布副本在本仓库中提交。

## 使用方式

1. 从上表对应的 GitHub 项目安装并配置 CLI。
2. 将 `skills/<name>` 复制或链接到 Agent 可发现的 skills 目录。
3. 确认 CLI 已加入 `PATH`，再显式调用 skill：

```text
$db-query 使用 uat profile 查询 logistics.t_waybill 中最近 20 条记录
$loki-query 使用 prod profile 查询最近 30 分钟内订单 252143 的异常日志
```

两个 skill 都必须显式调用，避免普通对话访问数据库或日志环境。数据库连接
信息和 Grafana Token 保留在各 CLI 的 profile 与环境变量中，不写入本仓库。

## 工作区结构

```text
skills/                         # 同步的发布副本
├── db-query/
│   ├── SKILL.md
│   └── agents/openai.yaml
└── loki-query/
    ├── SKILL.md
    └── agents/openai.yaml
db-cli/                         # 独立 Git worktree
└── .agents/skills/db-query/    # db-query 唯一事实来源
loki-query/                     # 独立 Git worktree
└── .agents/skills/loki-query/  # loki-query 唯一事实来源
```
