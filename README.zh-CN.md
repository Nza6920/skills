# me-skills

[English](README.md) | 简体中文

本仓库用于分发和协调 `db-query`、`loki-query`、`redis-query` Agent Skills。
数据库和 Loki 子项目维护各自的 CLI 与权威 Skill 源文件，本仓库在 `skills/`
下发布同步副本。Redis Skill 直接在本仓库维护，使用官方 `redis-cli`。

## 唯一事实来源

| Skill | 作用 | Skill 目录 | CLI 项目或文档 |
| --- | --- | --- | --- |
| `db-query` | 通过明确选择的 profile 执行受保护的 MySQL 只读查询。要求使用数据库限定表名、限制查询成本，并在访问 production 前批准完整且未变化的 profile 与 SQL。 | [`skills/db-query`](skills/db-query) | [Nza6920/db-cli](https://github.com/Nza6920/db-cli) |
| `loki-query` | 通过明确选择的 profile 查询 Grafana Loki。限制查询时间窗和次数，并将日志证据、推断与待核实事项分开报告。 | [`skills/loki-query`](skills/loki-query) | [Nza6920/grafana-loki-query-cli](https://github.com/Nza6920/grafana-loki-query-cli) |
| `redis-query` | 使用明确选择的本地环境，通过官方 CLI 执行受限的 Redis 只读查询。访问生产环境前，必须批准完整且未变化的目标与命令。 | [`skills/redis-query`](skills/redis-query) | [Redis 官方 CLI](https://redis.io/docs/latest/develop/tools/cli/) |

在当前工作区中，权威入口为：

```text
db-cli/.agents/skills/db-query/SKILL.md
loki-query/.agents/skills/loki-query/SKILL.md
skills/redis-query/SKILL.md
```

`db-cli/` 和 `loki-query/` 是被根仓库忽略的独立 Git worktree。应先在对应
子项目中修改 Skill，再将完整 Skill 目录同步到 `skills/`。源文件变更在子项目
中提交，发布副本在本仓库中提交。直接修改 `skills/redis-query/`，它没有独立
的 CLI 子项目。

## 使用方式

1. 按上表对应项目或官方文档安装并配置 CLI。Redis 每个环境使用独立的本地
   `~/.redis-env.<环境名>` 文件，参照[配置模板](skills/redis-query/references/connection.md)。
2. 将 `skills/<name>` 复制或链接到 Agent 可发现的 skills 目录。
3. 确认 CLI 已加入 `PATH`，再显式调用 skill：

```text
$db-query 使用 uat profile 查询 logistics.t_waybill 中最近 20 条记录
$loki-query 使用 prod profile 查询最近 30 分钟内订单 252143 的异常日志
$redis-query 使用 uat 环境查看 cache:order:252143 的类型、TTL 和受限内容预览
```

三个 skill 都必须显式调用，避免普通对话访问数据库或日志环境。连接信息和
认证信息保留在本地配置与环境变量中，不写入本仓库。

Redis 环境文件使用 `REDISCLI_AUTH`、`REDIS_HOST`、`REDIS_PORT`，并增加必填的
`REDIS_ENV_TYPE=production|non-production`。认证仅使用 `REDISCLI_AUTH` 中的完整
密码，原样保留 `@` 等字符，不拆分、不进行 URL 编码。可选配置包括 DB
（默认 0）、TLS 和 Cluster。认证配置缺失时，先引导用户在本地创建，验证通过后
再连接，密码不进入对话。默认最多执行十次扫描，使用 `COUNT 100`，最多展示
100 项，String 值最多预览 4 KiB。扫描 COUNT 是工作量提示，并非响应数量硬上限。

## 工作区结构

```text
skills/                         # 发布的 Skills
├── db-query/
│   ├── SKILL.md
│   └── agents/openai.yaml
├── loki-query/
│   ├── SKILL.md
│   └── agents/openai.yaml
└── redis-query/                 # 根仓库维护的唯一事实来源
    ├── SKILL.md
    ├── agents/openai.yaml
    └── references/
        ├── connection.md
        └── queries.md
db-cli/                         # 独立 Git worktree
└── .agents/skills/db-query/    # db-query 唯一事实来源
loki-query/                     # 独立 Git worktree
└── .agents/skills/loki-query/  # loki-query 唯一事实来源
```
