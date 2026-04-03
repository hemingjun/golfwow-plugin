# GolfWow Plugin

Claude Code Skill Plugin，通过 SSH + psql 直连查询和管理 GolfWow Supabase 数据库。

## 前提条件

- Tailscale 连接 NAS（100.89.220.1）
- SSH 访问：`ssh hemingjun@100.89.220.1`
- Claude Code 已授权 `Bash(ssh:*)` 权限

## Skills

| Skill | 用途 |
|-------|------|
| search | 数据查询 — 任意表查询、跨表 JOIN、聚合统计 |
| write | 数据写入 — INSERT/UPDATE/DELETE，含安全规则 |

## 执行方式

所有操作通过以下命令链执行：

```bash
ssh hemingjun@100.89.220.1 "docker exec supabase-db psql -U supabase_admin -d postgres -c 'SQL'"
```

## 后续扩展

- ops-manage — 行程/预订运营管理
- data-audit — 数据质量巡检
- report — 统计报表生成
