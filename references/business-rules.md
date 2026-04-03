# 业务规则

GolfWow 数据写入时必须遵守的业务规则。write skill 执行前必须参考此文件。

## 状态流转

### data_status（数据完善度）
```
补充中 → 待审核 → 已完善
```
不允许反向流转。

### booking_status（预订状态）
```
计划中 → 已预订 → 已确认
任意状态 → 已取消
```
不允许：已确认 → 计划中，已取消 → 已预订。

## 必填字段

| 表 | 必填字段 |
|---|---|
| 所有实体表 | name_en（或 name） |
| facility.golf_courses | club_id（必须属于某个球会） |
| ops.trips | project_type, project_date_start |
| finance.transactions | transaction_type, amount, currency, transaction_date |
| crm.customers | name_cn 或 (last_name + first_name) |
| link.asset_links | asset_id, entity_type, entity_id |

## 外键约束

所有外键默认 `ON DELETE RESTRICT`，即有下游依赖时不能删除。

**不能删除的场景**：
- golf_club 有关联 golf_courses 时
- trip 有关联 bookings/flights/timeline 时
- partner 有关联 contacts/meetings 时
- hotel/course 被 bookings 引用时
- bank_account 有关联 transactions 时

**关联表（link schema）例外**：`ON DELETE CASCADE`，删除主体时自动清理关联。

## 安全红线

1. **finance.transactions 禁止 DELETE** — 财务记录只增不删，错误记录通过新增冲正条目修正
2. **ops.trips 删除前必须确认** — 所有 bookings 和 flights 已取消或删除
3. **批量操作前必须确认影响行数** — 先 `SELECT COUNT(*)` 再执行
4. **枚举值必须校验** — 参考 enums.md，PostgreSQL 会拒绝非法枚举值并报错

## 产品派生链

```
route_templates → wholesale_packages → travel_plans → trips
```

上游修改不会自动同步下游。如果修改了 route_template，需要手动检查并更新对应的 wholesale_packages。

## 通用写入规范

- INSERT 始终加 `RETURNING *` 确认写入结果
- UPDATE 始终加 `WHERE id = 'uuid'` 精确匹配，避免全表更新
- UPDATE 始终加 `updated_at = now()`
- DELETE 始终加 `RETURNING *` 记录被删数据
- 中文枚举值用单引号：`'公共球场'`
