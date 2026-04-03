---
name: write
description: 写入 GolfWow 业务数据。当用户要添加、更新、修改、删除数据时使用。包含安全规则：DELETE 二次确认、先查后改、状态流转校验、枚举值校验。通过 SSH + psql 直连 Supabase PostgreSQL 执行 SQL。
---

# GolfWow Write

写入（INSERT / UPDATE / DELETE）GolfWow Supabase 数据库中的业务数据。

## 执行方式

通过 SSH 连接 NAS，在 Supabase PostgreSQL 容器中执行 SQL：

```bash
ssh hemingjun@100.89.220.1 "docker exec supabase-db psql -U supabase_admin -d postgres -c \"
  SQL_HERE
\""
```

复杂语句用 heredoc：

```bash
ssh hemingjun@100.89.220.1 << 'REMOTE'
docker exec supabase-db psql -U supabase_admin -d postgres << 'SQL'
  SQL_HERE
SQL
REMOTE
```

## 安全规则（必须遵守）

### 1. DELETE 必须二次确认
先 SELECT 展示将删除的记录，等用户明确确认后再执行 DELETE。绝不跳过确认。

### 2. 先查后改
UPDATE 前先 SELECT 当前值，展示变更对比（旧值 → 新值），确认后再执行。

### 3. 状态流转约束
- **booking_status**: 计划中 → 已预订 → 已确认（或任意状态 → 已取消）
- **data_status**: 补充中 → 待审核 → 已完善
- 不允许反向流转（如 已确认 → 计划中）
- UPDATE 的 WHERE 条件中包含当前状态，防止并发覆盖

### 4. 必填字段检查
INSERT 前确认必填字段已提供。参考 `references/business-rules.md`。

### 5. 枚举值校验
写入枚举字段前，参考 `references/enums.md` 确保值合法。

### 6. 外键检查
引用的外键记录必须存在。可用 SELECT 验证：
```sql
SELECT id FROM geo.regions WHERE id = 'target-uuid';
```

### 7. 批量操作确认
批量 UPDATE/DELETE 前先 `SELECT COUNT(*)` 确认影响行数，展示给用户确认。

## 写入模式

### INSERT
```sql
INSERT INTO schema.table (field1, field2, created_at, updated_at)
VALUES ('value1', 'value2', now(), now())
RETURNING *;
```
始终加 `RETURNING *` 确认写入结果。

### UPDATE
```sql
-- 第一步：查看当前值
SELECT id, field1, field2 FROM schema.table WHERE id = 'uuid';

-- 第二步：展示变更，用户确认后执行
UPDATE schema.table
SET field1 = 'new_value', updated_at = now()
WHERE id = 'uuid'
RETURNING *;
```

### DELETE
```sql
-- 第一步：展示将删除的记录
SELECT * FROM schema.table WHERE id = 'uuid';

-- 第二步：检查下游依赖
SELECT COUNT(*) FROM downstream.table WHERE foreign_key = 'uuid';

-- 第三步：用户确认后删除
DELETE FROM schema.table WHERE id = 'uuid' RETURNING *;
```

**finance.transactions 禁止 DELETE** — 错误记录通过新增冲正条目修正。

### 批量操作
```sql
-- 先确认影响行数
SELECT COUNT(*) FROM schema.table WHERE condition;

-- 用户确认后执行
UPDATE schema.table
SET field = 'new_value', updated_at = now()
WHERE condition
RETURNING id, field;
```

## 常用写入场景

### 更新数据状态
```sql
SELECT id, name_en, data_status FROM facility.golf_clubs WHERE id = 'uuid';
-- 确认后：
UPDATE facility.golf_clubs
SET data_status = '已完善', updated_at = now()
WHERE id = 'uuid' AND data_status = '待审核'
RETURNING id, name_en, data_status;
```

### 新增客户
```sql
INSERT INTO crm.customers (last_name, first_name, name_cn, email, country_id, data_status, created_at, updated_at)
VALUES ('Smith', 'John', '史密斯', 'john@example.com',
  (SELECT id FROM geo.countries WHERE name_en = 'Canada'),
  '补充中', now(), now())
RETURNING *;
```

### 更新预订状态
```sql
SELECT id, booking_status FROM ops.golf_bookings WHERE id = 'uuid';
-- 确认后：
UPDATE ops.golf_bookings
SET booking_status = '已预订', updated_at = now()
WHERE id = 'uuid' AND booking_status = '计划中'
RETURNING *;
```

### 新增关联
```sql
INSERT INTO link.course_hotel_link (course_id, hotel_id, sort_order)
VALUES ('course-uuid', 'hotel-uuid', 1);
```

## 参考文件

写入前读取这些文件确认规则：
- `references/business-rules.md` — 状态流转、必填字段、安全红线
- `references/enums.md` — 枚举值校验
- `references/schema-overview.md` — 表结构确认
