# Data Model (MVP) — TikTok Auto Ops (VN + TikTok Shop)

目标：围绕“内容工厂 → 人工发布闸口 → 数据复盘”的闭环，为 3C 小数码配件（越南站 TikTok Shop）提供可扩展的数据结构。

> 技术基线：Postgres + NestJS API + Python Worker（渲染）+ Next.js Console  
> 原则：结构化数据优先；允许 JSONB 承载可变字段；所有关键动作可追溯（audit）。

---

## 0. 约定（Conventions）

- 主键：`uuid`（推荐 `gen_random_uuid()`，需要 pgcrypto；或 uuid-ossp 也可）
- 时间：`timestamptz`
- 软删除：`deleted_at`（NULL 表示有效）
- 货币：整数 VND（`price_vnd` / `price_min_vnd` / `price_max_vnd`）
- 状态：`status` / `state` 字段使用枚举值（text + check 或 pg enum）
- 审核闸口：凡涉及发布/对外动作，一律记录 `approved_by`、`approved_at`
- JSONB：用于可扩展字段，但要配合 schema 约定（在 docs 中写清）

---

## 1. products（商品 / SKU）

**用途**：承载 TikTok Shop 越南站可售的 SKU 信息，以及用于脚本/素材/模板选择的“卖点画像”。

字段建议：
- `id` uuid pk
- `sku` text unique（内部 SKU）
- `tiktok_shop_product_id` text nullable（后续对接用）
- `name_vi` text not null
- `name_zh` text nullable
- `category` text not null（如 cap-sac / cu-sac / op-lung / hub / kinh-cuong-luc）
- `price_min_vnd` int nullable
- `price_max_vnd` int nullable
- `core_benefits` jsonb not null default '[]'（1–3条核心卖点）
- `proof_points` jsonb not null default '[]'（协议/材质/质保/认证等证据点）
- `compatibility` jsonb not null default '{}'（接口/机型/协议：PD/QC/TypeC/Lightning）
- `offer` jsonb not null default '{}'（赠品/免邮/质保）
- `landing_url` text nullable（TikTok Shop 商品链接）
- `compliance_flags` jsonb not null default '{}'（敏感承诺、禁用词、需复核）
- `status` text not null default 'draft'（draft|active|paused）
- `created_at` timestamptz not null
- `updated_at` timestamptz not null
- `deleted_at` timestamptz nullable

索引建议：
- unique: `(sku)`
- index: `(status, updated_at desc)`

---

## 2. product_assets（素材/资产库）

**用途**：管理素材、权属、标签、适配关系；为“自动混剪/渲染”提供检索。

字段建议：
- `id` uuid pk
- `product_id` uuid fk -> products.id nullable（也允许素材不绑定商品：通用背景/片头）
- `asset_type` text not null（video|image|audio|subtitle|project）
- `source_type` text not null（self_made|supplier|licensed|remix_reference）
- `uri` text not null（对象存储路径/本地路径/URL）
- `checksum` text nullable（sha256，用于去重）
- `duration_ms` int nullable
- `width` int nullable
- `height` int nullable
- `tags` jsonb not null default '{}'  
  建议 key：`shot_type`、`pace`、`style`、`use_case`、`language`、`contains_text`
- `rights` jsonb not null default '{}'  
  建议 key：`authorization_doc_uri`、`license_type`、`expires_at`
- `compliance_notes` text nullable
- `created_at` timestamptz not null
- `updated_at` timestamptz not null
- `deleted_at` timestamptz nullable

索引建议：
- index: `(product_id)`
- index: `((tags->>'style'))`（按需）
- index: `(asset_type, source_type)`

---

## 3. script_templates（脚本模板）

字段建议：
- `id` uuid pk
- `name` text not null
- `template_type` text not null（pain_solution|comparison|review|scenario|unboxing|faq|anti_scam|how_to）
- `language` text not null（vi|en|zh）
- `schema` jsonb not null（槽位定义：hook/pain/proof/cta + 字数/时长约束）
- `status` text not null default 'active'（active|disabled）
- `created_at` timestamptz
- `updated_at` timestamptz

---

## 4. scripts（脚本实例）

字段建议：
- `id` uuid pk
- `product_id` uuid fk -> products.id
- `template_id` uuid fk -> script_templates.id
- `language` text not null
- `content` jsonb not null（结构化段落，支持 timing hints）
- `hook_variant` text nullable
- `cta_variant` text nullable
- `created_at` timestamptz not null

索引建议：
- index: `(product_id, created_at desc)`

---

## 5. render_templates（视频模板定义，可选但推荐）

如果你希望模板可配置，而不是写死在 worker 里，建议加一张表：

- `id` uuid pk
- `code` text unique（T1_compare / T2_pain / T3_unboxing）
- `description` text
- `default_config` jsonb（字幕样式、片头、转场）
- `status` text（active|disabled）

---

## 6. renders（渲染任务与结果）

字段建议：
- `id` uuid pk
- `product_id` uuid fk -> products.id
- `script_id` uuid fk -> scripts.id
- `render_template_code` text not null（或 fk -> render_templates.code）
- `config` jsonb not null（素材选择、字幕风格、语音、音乐、封面、时长）
- `output_uri` text nullable
- `status` text not null（queued|running|succeeded|failed|canceled）
- `error` text nullable
- `created_at` timestamptz not null
- `updated_at` timestamptz not null

索引建议：
- index: `(status, created_at desc)`

---

## 7. accounts（账号）

字段建议：
- `id` uuid pk
- `handle` text not null unique
- `region` text not null default 'VN'
- `language` text not null default 'vi'
- `niche` text not null default '3c_accessories'
- `daily_cap` int not null default 3
- `publish_windows` jsonb not null default '[]'  
  例：`[{ "start":"11:30", "end":"13:30" }, { "start":"19:00", "end":"22:30" }]`
- `status` text not null default 'active'（active|paused）
- `created_at` timestamptz
- `updated_at` timestamptz

---

## 8. publish_tasks（排程 + 人工发布闸口）

字段建议：
- `id` uuid pk
- `account_id` uuid fk -> accounts.id
- `render_id` uuid fk -> renders.id
- `scheduled_at` timestamptz not null
- `state` text not null（pending_review|approved|published|skipped|failed）
- `approved_by` text nullable
- `approved_at` timestamptz nullable
- `published_video_url` text nullable
- `published_video_id` text nullable
- `failure_reason` text nullable
- `created_at` timestamptz
- `updated_at` timestamptz

索引建议：
- index: `(account_id, scheduled_at)`
- index: `(state, scheduled_at)`

---

## 9. video_metrics（指标采样）

字段建议：
- `id` uuid pk
- `publish_task_id` uuid fk -> publish_tasks.id
- `sampled_at` timestamptz not null
- `views` int not null default 0
- `likes` int not null default 0
- `comments` int not null default 0
- `shares` int not null default 0
- `avg_watch_time_ms` int nullable
- `completion_rate` numeric nullable
- `product_clicks` int nullable
- `orders` int nullable

索引建议：
- unique: `(publish_task_id, sampled_at)`

---

## 10. experiments（A/B 测试登记，可选）

字段建议：
- `id` uuid pk
- `dimension` text（hook|cta|template|caption|cover）
- `variant_a` text
- `variant_b` text
- `start_at` timestamptz
- `end_at` timestamptz
- `success_metric` text（views_1h|ctr|orders）
