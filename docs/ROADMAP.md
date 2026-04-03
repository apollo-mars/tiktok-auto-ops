# Roadmap — TikTok Auto Ops（VN + TikTok Shop）

技术基线：NestJS API + Next.js Console + Postgres + Python Render Worker（Docker）。

> MVP 14 天目标：跑通 “SKU → 脚本 → 渲染 → 人工审核发布 → 指标回填 → 复盘建议”。

---

## MVP（14 days）

### Day 1–2：Repo Bootstrap（协作开工）
- 目录骨架：
  - `services/api`（NestJS）
  - `apps/console`（Next.js）
  - `services/worker`（Python）
- 根目录 `docker-compose.yml` 一键启动 Postgres + api + console + worker
- `.env.example`（DATABASE_URL、OBJECT_STORE_URI、TTS_PROVIDER_KEY 等）
- 基础代码规范：
  - TS：eslint + prettier
  - Python：ruff + black
- CI（可选）：lint + unit test

**验收**
- `docker compose up` 后 console/api 能访问
- api 能连上 postgres（healthcheck）

---

### Day 3–5：Product Hub + Asset Library（可用的基础数据）
- Postgres migrations：
  - `products`
  - `product_assets`
- NestJS API：
  - products CRUD（含软删除）
  - assets CRUD（含上传/登记、权属字段）
- Console：
  - 商品列表/详情/编辑
  - 素材上传/登记页面（先做登记也可以）

**验收**
- 能录入 20 个 SKU + 每个 SKU 3–10 个素材
- 支持按标签检索素材

---

### Day 6–7：Script Engine（脚本模板 + 多变体）
- script_templates 表 + API
- scripts 表 + 生成接口：
  - 输入：product_id + template_type + 变体数量
  - 输出：结构化 JSON 脚本（含 hook/pain/proof/cta）
- Console：
  - 模板维护页
  - 一键生成脚本变体 + 预览

**验收**
- 单 SKU 可生成 ≥10 条越南语脚本变体
- 支持禁用模板、保留历史

---

### Day 8–10：Render Worker（Python，3模板起步）
- renders 表 + API
- worker：
  - 轮询/订阅渲染任务（queued → running → succeeded/failed）
  - 模板：对比 / 痛点 / 开箱
  - 字幕：从 scripts.content 自动生成（可先简化）
  - TTS：stub + 可插拔接口
- 输出视频：
  - MVP 可先本地路径；后续再接 MinIO/S3

**验收**
- 任意脚本 + 素材可稳定渲染输出 mp4
- 失败可追踪 error 日志

---

### Day 11–12：Scheduler + Human Publish Gate（核心风控）
- accounts 表 + API
- publish_tasks 表 + API：
  - 创建排程任务
  - 审核（Approve）
  - 发布回填（video_url/video_id）
- Console：
  - 账号配置（daily cap / publish windows）
  - 待审核/待发布任务列表
  - 一键“已发布回填”（MVP手工）

**验收**
- 发布前必须人工 approve
- 每个账号每日不超过 daily_cap（逻辑约束）

---

### Day 13–14：Analytics + Feedback（复盘闭环）
- video_metrics 表 + API（MVP允许手动录入）
- 复盘规则引擎（先规则后模型）：
  - hook弱/结尾弱/素材疲劳 的标签输出
- Console：
  - 视频表现列表 + 复盘建议

**验收**
- 能对过去 7 天视频输出可行动建议（下一轮该改什么）

---

## 下一阶段（Scale-up）
- 自动化指标采集（对接 TikTok 可用方式/人工导入CSV）
- 模板市场化：模板包版本管理、模板评估
- 多账号矩阵策略：账号分层、内容分层、素材去重策略
- 更强合规：素材权属审计、敏感词/宣称校验、合规审查工作流
