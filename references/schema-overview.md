# GolfWow 数据库 Schema 速查手册

> 自部署 Supabase (PostgreSQL)。共 10 个 Schema，按业务域组织。
> 所有查询需 authenticated 身份（RLS 策略）。

---

## Schema 概览

| Schema | 用途 |
|--------|------|
| geo | 地理层级（洲 → 国家 → 省/州 → 地区） |
| facility | 设施（球会、球场、酒店、机场） |
| info | 行业信息（设计师、赛事） |
| media | 媒体素材元数据 |
| web | 网站展示层（profiles，触发器自动同步） |
| link | 多对多关联表（_link 后缀 + asset_links） |
| crm | CRM（供应商、联系人、客户、展会、会议） |
| product | 产品（路线模板、批发包、每日计划、出行计划） |
| ops | 运营（行程、时间线、里程碑、预订、航班） |
| finance | 财务（交易、银行账户） |

---

## 各 Schema 详情

### geo — 地理层级

**geo.continents**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 (uuidv7) |
| notion_id | text | Notion 同步 ID（唯一） |
| name_en | text | 英文名 |
| name_cn | text | 中文名 |
| description_en | text | 英文描述 |
| description_cn | text | 中文描述 |
| data_status | enum | 补充中 / 待审核 / 已完善 |
| created_at / updated_at | timestamptz | 时间戳 |

**geo.countries**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name_en / name_cn | text | 双语名称 |
| continent_id | uuid | FK → geo.continents |
| recommended_months | text | 推荐出行月份 |
| local_temperature | text | 当地气温说明 |
| description_en / description_cn | text | 双语描述 |
| data_status | enum | 数据状态 |

**geo.provinces**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name_en / name_cn | text | 双语名称 |
| country_id | uuid | FK → geo.countries |
| description_en / description_cn | text | 双语描述 |
| data_status | enum | 数据状态 |

**geo.regions**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name_en / name_cn | text | 双语名称 |
| province_id | uuid | FK → geo.provinces |
| description_en / description_cn | text | 双语描述 |
| data_status | enum | 数据状态 |

---

### facility — 设施

**facility.golf_clubs**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name_en / name_cn | text | 球会双语名称 |
| region_id | uuid | FK → geo.regions |
| address | text | 地址 |
| phone | text | 电话 |
| website | text | 官网 |
| google_place_id | text | Google Places ID |
| google_rating | numeric | Google 评分 |
| description_en / description_cn | text | 双语描述 |
| notes | text | 内部备注 |
| data_status | enum | 数据状态 |

**facility.golf_courses**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name_en / name_cn | text | 球场双语名称 |
| club_id | uuid | FK → facility.golf_clubs |
| holes | int | 洞数 |
| par | int | 标准杆 |
| length_yards | int | 球场长度（码） |
| slope | int | 坡度指数 |
| course_rating | numeric | 球场评分 |
| open_year | int | 开放年份 |
| access_type | enum | 公共球场 / 半会员制 / 私人球场 |
| course_features | enum[] | 球场特色标签 |
| course_prestige | enum[] | 世界百佳 / 国家百佳 / 区域知名 / 普通球场 / 名门球场 |
| terrain_style | enum[] | 林克斯 / 森林园景 / 沙丘 / 山地 |
| recommended_rating | numeric | 推荐评分 |
| description_en / description_cn | text | 双语描述 |

**facility.hotels**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name_en / name_cn | text | 酒店双语名称 |
| region_id | uuid | FK → geo.regions |
| address | text | 地址 |
| phone | text | 电话 |
| website | text | 官网 |
| star_rating | int | 星级 |
| open_year | int | 开业年份 |
| hotel_type | enum | 酒店类型 |
| price_range | enum | 价格区间 |
| check_in_time / check_out_time | text | 入住/退房时间 |
| breakfast_start / breakfast_end / breakfast_weekend | text | 早餐时间 |
| description_en / description_cn | text | 双语描述 |
| notes | text | 内部备注 |
| data_status | enum | 数据状态 |

**facility.airports**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| iata_code | text | IATA 机场代码 |
| name_en / name_cn | text | 双语名称 |
| region_id | uuid | FK → geo.regions |
| address | text | 地址 |
| data_status | enum | 数据状态 |

---

### info — 行业信息

**info.designers**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| name_en / name_cn | text | 设计师双语名称 |
| introduction_en / introduction_cn | text | 双语介绍 |
| data_status | enum | 数据状态 |

**info.tournaments**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| name_en / name_cn | text | 赛事双语名称 |
| description_en / description_cn | text | 双语描述 |
| data_status | enum | 数据状态 |

---

### media — 媒体素材

**media.assets**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键（前 16 位用于文件命名） |
| file_path | text | Storage 内路径 |
| bucket | text | media / media-internal / assets |
| file_name | text | 原始文件名 |
| display_name | text | 展示名称 |
| media_type | enum | photo / video |
| mime_type | text | MIME 类型 |
| file_size | bigint | 文件大小（字节） |
| width / height | int | 图片尺寸 |
| duration_seconds | int | 视频时长 |
| description_en / description_cn | text | 双语描述 |
| tags | text[] | 标签 |
| uploaded_by | uuid | 上传用户 |
| source | text | 来源说明 |
| notion_id | text | Notion ID |
| data_status | enum | 数据状态 |

公开访问 URL 格式：`https://supabase.golfwow.com/storage/v1/object/public/media/{file_path}`

---

### web — 网站展示层

**web.profiles**（触发器自动同步，不需手动写入）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| entity_type | enum | 实体类型（golf_club / hotel / ...） |
| entity_id | uuid | 对应实体 ID |
| slug | text | URL slug |
| publishable | bool | 是否可发布 |
| title_en / title_cn | text | 双语标题 |
| meta_description_en / meta_description_cn | text | SEO 描述 |
| metadata | jsonb | 额外元数据 |
| image_urls | text[] | 图片 URL 列表 |

---

### link — 关联表

所有关联表均为多对多，无独立业务逻辑。

| 表名 | 关联关系 |
|------|---------|
| link.asset_links | asset_id → entity_type + entity_id（素材与任意实体） |
| link.club_partner_link | golf_clubs ↔ crm.partners |
| link.course_designer_link | golf_courses ↔ info.designers（含 role、sort_order） |
| link.course_hotel_link | golf_courses ↔ hotels |
| link.course_tournament_link | golf_courses ↔ info.tournaments（含 years 历届） |
| link.customer_link | customers 自关联（同行客人） |
| link.hotel_airport_link | hotels ↔ airports（含 distance_km、drive_minutes） |
| link.hotel_partner_link | hotels ↔ crm.partners |
| link.route_country_link | route_templates ↔ geo.countries |
| link.route_course_link | route_templates ↔ golf_courses（含 notes） |
| link.route_hotel_link | route_templates ↔ hotels（含 notes） |
| link.travel_plan_country_link | travel_plans ↔ geo.countries |
| link.travel_plan_course_link | travel_plans ↔ golf_courses |
| link.travel_plan_hotel_link | travel_plans ↔ hotels |
| link.trip_customer_link | ops.trips ↔ crm.customers |

**link.asset_links 字段**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| asset_id | uuid | FK → media.assets |
| entity_type | enum | 实体类型 |
| entity_id | uuid | 实体 ID |
| role | text | 用途角色（cover / gallery / ...） |
| sort_order | int | 排序 |

---

### crm — 客户关系管理

**crm.partners**（供应商）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name | text | 供应商名称 |
| partner_type | enum | 供应商类型 |
| website | text | 官网 |
| region_id | uuid | FK → geo.regions |
| description_en / description_cn | text | 双语描述 |
| notes | text | 备注 |
| data_status | enum | 数据状态 |

**crm.contacts**（联系人）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name_en / name_cn | text | 双语姓名 |
| email | text | 邮箱 |
| phone | text | 电话 |
| position | text | 职位 |
| partner_id | uuid | FK → crm.partners |
| region_id | uuid | FK → geo.regions |
| notes | text | 备注 |
| data_status | enum | 数据状态 |

**crm.customers**（终端客户）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| last_name / first_name | text | 姓 / 名 |
| nickname | text | 昵称 |
| name_cn | text | 中文姓名 |
| tags | enum[] | 客户标签 |
| country_id | uuid | FK → geo.countries |
| phone | text | 电话 |
| email | text | 邮箱 |
| wechat | text | 微信 |
| handicap | numeric | 差点 |
| birthday | date | 生日 |
| address | text | 地址 |
| notes | text | 备注 |
| service_needs | text | 服务需求 |
| dietary_notes | text | 饮食备注 |
| data_status | enum | 数据状态 |

**crm.events**（展会）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name_en / name_cn | text | 双语名称 |
| event_year | int | 年份 |
| location | text | 地点 |
| region_id | uuid | FK → geo.regions |
| description_en / description_cn | text | 双语描述 |
| data_status | enum | 数据状态 |

**crm.meetings**（会议记录）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name | text | 会议名称 |
| event_id | uuid | FK → crm.events |
| partner_id | uuid | FK → crm.partners |
| contact_id | uuid | FK → crm.contacts |
| agreement_status | enum | 合作协议状态 |
| cooperation_level | text | 合作级别 |
| notes | text | 备注 |

---

### product — 产品

**product.route_templates**（路线模板，产品派生链起点）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name | text | 模板名称 |
| recommended_days | int | 推荐天数 |
| recommended_rounds | int | 推荐轮次 |
| suitable_months | int[] | 适合月份 |
| target_audience | text[] | 目标客群 |
| travel_style | text[] | 出行风格 |
| weather_info | text | 天气说明 |
| notes | text | 备注 |
| data_status | enum | 数据状态 |

**product.wholesale_packages**（批发包）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name | text | 包名 |
| route_template_id | uuid | FK → route_templates |
| currency | enum | CAD / USD / CNY |
| price_valid_until | date | 定价有效期 |
| min_group_size | int | 最小成团人数 |
| cost_golfer / cost_non_golfer | numeric | 成本价（球手/非球手） |
| markup_golfer / markup_non_golfer | numeric | 加价（球手/非球手） |
| markup_retail | numeric | 零售加价 |
| single_supplement | numeric | 单人附加费 |
| golf_inclusions | text[] | 球场包含内容 |
| hotel_inclusions | text[] | 酒店包含内容 |
| extra_inclusions | text[] | 其他包含内容 |
| notes_* | text | 各类备注字段 |
| data_status | enum | 数据状态 |

**product.wholesale_daily_plans**（批发包每日计划）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| day_label | text | 日期标签（如 Day 1） |
| package_id | uuid | FK → wholesale_packages |
| course_id | uuid | FK → facility.golf_courses |
| hotel_id | uuid | FK → facility.hotels |
| description | text | 当日描述 |
| notes | text | 备注 |
| sort_order | int | 排序 |

**product.travel_plans**（出行计划，面向客户）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name | text | 计划名称 |
| name_cn / name_en | text | 双语名称 |
| route_template_id | uuid | FK → route_templates |
| wholesale_package_id | uuid | FK → wholesale_packages |
| departure_date | date | 出发日期 |
| duration_days | int | 行程天数 |
| starting_price | numeric | 起步价 |
| currency | enum | 货币 |
| remaining_slots | int | 剩余名额 |
| is_featured | bool | 是否精选 |
| hotel_stars | int[] | 酒店星级 |
| description_cn / description_en | text | 双语描述 |
| itinerary_text | text | 行程文本 |
| other_activities | text | 其他活动 |
| meal_info | text | 餐食说明 |
| services | text[] | 服务项目 |
| data_status | enum | 数据状态 |

---

### ops — 运营

**ops.trips**（行程，核心运营实体）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name | text | 行程名称 |
| project_date_start / project_date_end | date | 行程起止日期 |
| project_type | enum | 私人定制 / 大团 / 订场服务 / 散客 |
| headcount | int | 人数 |
| project_planned | bool | 是否已规划 |
| finance_recorded | bool | 是否已记账 |
| folder_url | text | Google Drive 文件夹链接 |
| googledrive_id | text | Google Drive ID |
| notes | text | 备注 |
| data_status | enum | 数据状态 |

**ops.trip_timeline**（时间线）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| trip_id | uuid | FK → ops.trips |
| event_date | date | 事件日期 |
| sort_order | int | 排序 |
| event_type | enum | 事件类型 |
| booking_type | enum | 预订类型 |
| booking_id | uuid | 关联预订 ID |
| title | text | 标题 |
| notes | text | 备注 |

**ops.trip_milestones**（里程碑）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| trip_id | uuid | FK → ops.trips |
| milestone_key | text | 里程碑键名 |
| completed | bool | 是否完成 |
| completed_at | timestamptz | 完成时间 |
| note | text | 备注 |

**ops.golf_bookings**（球场预订）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| trip_id | uuid | FK → ops.trips |
| course_id | uuid | FK → facility.golf_courses |
| play_date | date | 打球日期 |
| tee_time | time | 开球时间 |
| players | int | 球手数 |
| carts | int | 球车数 |
| booking_status | enum | 计划中 / 已预订 / 已确认 / 已取消 |
| voucher | text | 凭证号 |
| booked_from | text | 预订来源 |
| cancel_deadline | date | 取消截止日 |
| cost | numeric | 费用 |
| paid | numeric | 已付金额 |
| supplier_contact | text | 供应商联系人 |
| last_contact_date | date | 最近联系日期 |
| payment_deadline | date | 付款截止日 |
| final_confirmed | bool | 是否最终确认 |
| notes | text | 备注 |

**ops.hotel_bookings**（酒店预订）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| trip_id | uuid | FK → ops.trips |
| hotel_id | uuid | FK → facility.hotels |
| check_in / check_out | date | 入住/退房日期 |
| rooms | int | 房间数 |
| room_type | text | 房型 |
| room_class | enum | 房间等级 |
| bed_type | enum | 床型 |
| room_view | enum | 景观 |
| booking_status | enum | 预订状态 |
| voucher | text | 凭证号 |
| booked_from | text | 预订来源 |
| cancel_deadline | date | 取消截止日 |
| cost | numeric | 费用 |
| paid | numeric | 已付金额 |
| supplier_contact | text | 供应商联系人 |
| last_contact_date | date | 最近联系日期 |
| payment_deadline | date | 付款截止日 |
| final_confirmed | bool | 是否最终确认 |
| notes | text | 备注 |

**ops.transport_bookings**（交通预订）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| trip_id | uuid | FK → ops.trips |
| transport_type | enum | 交通类型 |
| service_date | date | 服务日期 |
| pickup_time | time | 接送时间 |
| from_location / to_location | text | 起点/终点 |
| passengers | int | 乘客数 |
| luggage_count | int | 行李件数 |
| vehicle_type | text | 车型 |
| booking_status | enum | 预订状态 |
| cost | numeric | 费用 |
| paid | numeric | 已付金额 |
| voucher | text | 凭证号 |
| booked_from | text | 预订来源 |
| cancel_deadline | date | 取消截止日 |
| supplier_contact | text | 供应商联系人 |
| last_contact_date | date | 最近联系日期 |
| payment_deadline | date | 付款截止日 |
| final_confirmed | bool | 是否最终确认 |
| notes | text | 备注 |

**ops.flights**（航班）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| trip_id | uuid | FK → ops.trips |
| airline | text | 航空公司 |
| flight_number | text | 航班号 |
| departure_date / departure_time | date/time | 起飞日期/时间 |
| arrival_date / arrival_time | date/time | 到达日期/时间 |
| origin / destination | text | 出发地/目的地 |
| passengers | int | 乘客数 |
| booking_status | enum | 预订状态 |
| cost | numeric | 费用 |
| paid | numeric | 已付金额 |
| booked_from | text | 预订来源 |
| voucher | text | 凭证号 |
| supplier_contact | text | 供应商联系人 |
| last_contact_date | date | 最近联系日期 |
| payment_deadline | date | 付款截止日 |
| final_confirmed | bool | 是否最终确认 |
| notes | text | 备注 |

---

### finance — 财务

**finance.bank_accounts**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| name | text | 账户名称 |
| initial_balance | numeric | 期初余额 |
| notes | text | 备注 |

**finance.transactions**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | uuid | 主键 |
| notion_id | text | Notion ID |
| transaction_no | text | 流水号 |
| transaction_date | date | 交易日期 |
| transaction_type | enum | 收入 / 支出 / 代收 / 代付 / commission |
| bill_categories | enum[] | 账单分类 |
| amount | numeric | 金额 |
| currency | enum | CAD / USD / CNY |
| payment_method | enum | 支付方式 |
| is_settled | bool | 是否结清 |
| trip_id | uuid | FK → ops.trips |
| bank_account_id | uuid | FK → finance.bank_accounts |
| counterparty_id | uuid | 交易对手 ID |
| counterparty_type | enum | 交易对手类型 |
| notes | text | 备注 |

---

## 常用视图

| Schema | 视图名 | 用途 |
|--------|--------|------|
| crm | contacts_view | 联系人（含供应商名称） |
| crm | meetings_view | 会议记录（含展会、供应商、联系人） |
| crm | partners_view | 供应商（含地区信息） |
| geo | countries_view | 国家（含大洲名称） |
| geo | provinces_view | 省/州（含国家名称） |
| geo | regions_view | 地区（含省/国家名称） |
| geo | regions_summary_view | 地区摘要（含设施数量统计） |
| facility | golf_clubs_view | 球会（含地区信息） |
| facility | golf_courses_view | 球场（含球会、地区信息） |
| facility | golf_courses_full_view | 球场完整视图（含设计师、赛事） |
| facility | hotels_view | 酒店（含地区信息） |
| finance | customer_summary | 客户财务汇总 |
| finance | trip_summary | 行程财务汇总 |
| link | club_hotel_view | 球会-酒店关联 |
| link | club_partner_view | 球会-供应商关联 |
| link | course_designer_view | 球场-设计师关联 |
| link | course_hotel_view | 球场-酒店关联 |
| link | course_tournament_view | 球场-赛事关联 |
| link | hotel_airport_view | 酒店-机场距离 |
| link | hotel_partner_view | 酒店-供应商关联 |
| link | route_country_view | 路线-国家关联 |
| link | route_course_view | 路线-球场关联 |
| link | route_hotel_view | 路线-酒店关联 |
| ops | golf_bookings_view | 球场预订（含球场、行程名） |
| ops | hotel_bookings_view | 酒店预订（含酒店、行程名） |
| ops | trip_customers_view | 行程客户列表 |
| ops | trip_timeline_view | 行程时间线（含预订详情） |
| product | wholesale_daily_plans_view | 批发包每日计划 |
| product | wholesale_daily_plans_detail | 批发包每日计划详情（含球场、酒店信息） |

---

## 外键关系

### 地理层级（自上而下）
```
geo.continents
  └── geo.countries (continent_id)
        └── geo.provinces (country_id)
              └── geo.regions (province_id)
```

### 设施与地理
```
geo.regions ←── facility.golf_clubs (region_id)
            ←── facility.hotels (region_id)
            ←── facility.airports (region_id)
            ←── crm.partners (region_id)
            ←── crm.events (region_id)

facility.golf_clubs ←── facility.golf_courses (club_id)
```

### 产品派生链
```
product.route_templates
  └── product.wholesale_packages (route_template_id)
        └── product.wholesale_daily_plans (package_id)
  └── product.travel_plans (route_template_id)
        └── [also] wholesale_packages (wholesale_package_id)
              └── ops.trips（派生运营）
```

### 运营关联
```
ops.trips ←── ops.trip_timeline (trip_id)
          ←── ops.trip_milestones (trip_id)
          ←── ops.golf_bookings (trip_id)
          ←── ops.hotel_bookings (trip_id)
          ←── ops.transport_bookings (trip_id)
          ←── ops.flights (trip_id)
          ←── link.trip_customer_link (trip_id)
          ←── finance.transactions (trip_id)
```

### CRM 关联
```
crm.partners ←── crm.contacts (partner_id)
             ←── crm.meetings (partner_id)
             ←── link.club_partner_link
             ←── link.hotel_partner_link

crm.events ←── crm.meetings (event_id)
geo.countries ←── crm.customers (country_id)
```

### 素材关联
```
media.assets ←── link.asset_links (asset_id)
link.asset_links.entity_id → 任意实体（通过 entity_type 区分）
```

---

## 关键枚举值速查

| 枚举 | 取值 |
|------|------|
| data_status | 补充中 / 待审核 / 已完善 |
| access_type | 公共球场 / 半会员制 / 私人球场 |
| course_prestige | 世界百佳 / 国家百佳 / 区域知名 / 普通球场 / 名门球场 |
| terrain_style | 林克斯 / 森林/园景 / 沙丘 / 山地 |
| project_type | 私人定制 / 大团 / 订场服务 / 散客 |
| booking_status | 计划中 / 已预订 / 已确认 / 已取消 |
| transaction_type | 收入 / 支出 / 代收 / 代付 / commission |
| currency | CAD / USD / CNY |
