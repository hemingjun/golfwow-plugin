---
description: 查询 GolfWow 业务数据（球场、酒店、行程、客户等）
argument-hint: 查询内容
---

使用 golfwow search skill 查询 GolfWow Supabase 数据库。

将 `$ARGUMENTS` 作为自然语言查询意图，例如 "所有公共球场"、"2026年行程统计"、"张三的预订"。

执行流程：
1. 读取 `references/schema-overview.md` 确定目标 schema 和表
2. 构造 SQL 查询（优先使用已有视图）
3. 通过 `ssh hemingjun@100.89.220.1 "docker exec supabase-db psql -U supabase_admin -d postgres -c 'SQL'"` 执行
4. 格式化结果返回

规范：
- 始终指定 schema：`schema.table`
- 大结果集加 LIMIT（默认 50）
- 中文枚举值用单引号
- 模糊搜索用 ILIKE
- 优先使用视图避免复杂 JOIN
