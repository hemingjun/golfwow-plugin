---
description: 写入 GolfWow 业务数据（新增、更新、删除）
argument-hint: 写入操作描述
---

使用 golfwow write skill 写入 GolfWow Supabase 数据库。

将 `$ARGUMENTS` 作为自然语言写入意图，例如 "新增客户张三"、"把XX球场状态改为已完善"、"取消XX预订"。

执行流程：
1. 读取 `references/schema-overview.md` 确认目标表和字段
2. 读取 `references/business-rules.md` 确认规则约束
3. 读取 `references/enums.md` 校验枚举值
4. 构造 SQL 并通过 `ssh hemingjun@100.89.220.1 "docker exec supabase-db psql -U supabase_admin -d postgres -c 'SQL'"` 执行

安全规则（必须遵守）：
- DELETE 必须先 SELECT 展示记录，等用户确认后再执行
- UPDATE 必须先查当前值，展示变更对比
- 状态流转：booking_status 只能 计划中→已预订→已确认（或→已取消）
- INSERT 始终加 RETURNING *
- finance.transactions 禁止 DELETE
