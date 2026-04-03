---
name: search
description: 查询 GolfWow 业务数据。当用户要查数据、找信息、统计数量、列出记录时使用。支持任意 schema/table 查询、跨表 JOIN、聚合统计。通过 SSH + psql 直连 Supabase PostgreSQL 执行 SQL。
---

# GolfWow Search

查询 GolfWow Supabase 数据库中的业务数据。

## 执行方式

通过 SSH 连接 NAS，在 Supabase PostgreSQL 容器中执行 SQL：

```bash
ssh hemingjun@100.89.220.1 "docker exec supabase-db psql -U supabase_admin -d postgres -c \"
  SQL_HERE
\""
```

复杂查询（避免转义问题）用 heredoc：

```bash
ssh hemingjun@100.89.220.1 << 'REMOTE'
docker exec supabase-db psql -U supabase_admin -d postgres << 'SQL'
  SQL_HERE
SQL
REMOTE
```

## 工作流

1. 理解用户意图
2. 读取 `references/schema-overview.md` 确定目标 schema 和表
3. 优先使用已有视图（见下方列表）
4. 构造 SQL 查询
5. 通过 ssh + psql 执行
6. 格式化结果返回给用户

## 查询规范

- 始终指定 schema：`schema.table`（如 `facility.golf_clubs`）
- 大结果集加 `LIMIT`（默认 50）
- 中文字段值用单引号：`WHERE access_type = '公共球场'`
- 模糊搜索用 ILIKE：`WHERE name_en ILIKE '%keyword%' OR name_cn ILIKE '%keyword%'`
- 统计用 `COUNT` / `SUM` + `GROUP BY`
- 排序默认 `ORDER BY created_at DESC`

## 常用视图

优先使用视图，避免手写复杂 JOIN：

| 视图 | 用途 |
|------|------|
| `facility.golf_courses_full_view` | 球场完整信息（含球会、地区） |
| `facility.golf_clubs_view` | 球会信息（含地区） |
| `facility.hotels_view` | 酒店信息（含地区） |
| `ops.trip_timeline_view` | 行程时间线（含预订详情） |
| `ops.trip_customers_view` | 行程客户列表 |
| `ops.golf_bookings_view` | 高尔夫预订（含球场名） |
| `ops.hotel_bookings_view` | 酒店预订（含酒店名） |
| `finance.trip_summary` | 行程财务汇总 |
| `finance.customer_summary` | 客户消费汇总 |
| `product.wholesale_daily_plans_detail` | 批发每日计划（含球场酒店名） |
| `geo.regions_summary_view` | 地区汇总（含球场酒店数量） |

## 查询模式

### 列出记录
```sql
SELECT id, name_en, name_cn, data_status
FROM facility.golf_clubs
ORDER BY name_en
LIMIT 50;
```

### 搜索
```sql
SELECT id, name_en, name_cn
FROM facility.golf_clubs
WHERE name_en ILIKE '%keyword%' OR name_cn ILIKE '%keyword%';
```

### 按枚举过滤
```sql
SELECT id, name_en, access_type
FROM facility.golf_courses
WHERE access_type = '公共球场';
```

### 数组枚举过滤
```sql
SELECT id, name_en, course_prestige
FROM facility.golf_courses
WHERE '世界百佳' = ANY(course_prestige);
```

### 统计
```sql
SELECT project_type, COUNT(*)
FROM ops.trips
GROUP BY project_type
ORDER BY count DESC;
```

### 跨表查询
```sql
SELECT t.name, t.project_type, c.name_cn as customer
FROM ops.trips t
JOIN link.trip_customer_link l ON l.trip_id = t.id
JOIN crm.customers c ON c.id = l.customer_id
WHERE t.project_type = '私人定制';
```

### 使用视图
```sql
SELECT * FROM facility.golf_courses_full_view
WHERE data_status = '已完善'
LIMIT 20;
```

### 日期范围
```sql
SELECT * FROM ops.trips
WHERE project_date_start BETWEEN '2026-01-01' AND '2026-12-31'
ORDER BY project_date_start;
```

## 参考文件

查询前读取这些文件确定正确的表和字段：
- `references/schema-overview.md` — 表结构速查
- `references/enums.md` — 枚举值参考
