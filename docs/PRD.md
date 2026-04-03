# TikTok Auto Ops — PRD（越南 | TikTok Shop | 3C小数码配件）

> 目标：用“自动化内容工厂 + 排程 + 数据复盘”的方式，从 1–2 个账号起步，稳定扩展到多账号矩阵。
> 成交链路：TikTok Shop 越南站（VN）。

## 1. 背景与目标

### 1.1 业务背景
你拥有 3C 小数码配件货源（充电线/充电头/保护壳/钢化膜/USB Hub 等），希望在越南市场通过 TikTok 内容带货，采用“混剪 + 二创”形式快速起量，并尽可能自动化运营。

### 1.2 产品目标（MVP）
在 14 天内打通闭环：
- 自动生成脚本与多版本视频（可审核候选池）
- 自动排程分发到账号（**人工确认发布闸口**）
- 发布后采集核心指标（至少播放/互动/留存代理指标）
- 自动复盘并给出下一轮“改钩子/改模板/改CTA/换素材”的建议

### 1.3 非目标（明确安全边界）
- 不做无人值守自动登录/自动发帖/自动刷互动
- 不做绕过平台安全机制的行为
- 不承诺任何销量/爆款结果

## 2. 用户与场景

### 2.1 核心用户
- 运营/老板（你）：选品、审核、发布确认、复盘决策
- 内容/剪辑协作者：维护模板、素材、字幕风格
- 开发协作者：API、worker、console、数据分析

### 2.2 核心场景（从1–2号到矩阵）
1) 录入/导入 SKU + 卖点 + 兼容机型 + TikTok Shop 商品链接  
2) 自动生成 5–20 个脚本变体（越南语为主，可扩展中/英）  
3) 自动从素材库选素材 → 使用模板渲染成 15/25/35s 三版本视频  
4) 运营在控制台审核：勾选通过 → 进入发布队列  
5) 到点提醒人工发布（或人工复制粘贴标题、选择视频文件）  
6) 回填 video_id/url → 系统开始采集数据并复盘

## 3. 功能范围（MVP）

### 3.1 Product Hub（商品中心）
- SKU 管理：名称（vi/zh）、类目、价格区间、卖点、兼容清单、优惠信息
- TikTok Shop 链接：landing_url
- 合规标签：敏感承诺/禁用词/需人工复核标记

### 3.2 Asset Library（素材库）
- 素材类型：视频/图片/音频/字幕/工程文件
- 权属/授权信息：来源（自制/供应商/授权）、证明文件链接、过期时间
- 标签体系：镜头类型、节奏、风格、适配 SKU、适配模板

### 3.3 Script Engine（脚本引擎）
- 8类带货结构模板（痛点解决/对比测评/防坑科普/场景/开箱细节/FAQ…）
- 自动生成多变体：hook、证据点、CTA 组合
- 输出结构化 JSON（便于渲染 worker 调用）

### 3.4 Video Composer Worker（Python + FFmpeg/MoviePy）
- 模板驱动渲染：至少 3 套（对比/痛点/开箱）
- 字幕：自动生成 + 关键词高亮
- TTS：可插拔（先 stub，后接入可用服务）
- 生成结果入库（renders）+ 输出视频存储（本地/对象存储均可）

### 3.5 Scheduler + Human Publish Gate（排程与人工发布闸口）
- 多账号配置：每日上限、发布时段窗口
- 发布任务队列：pending_review → approved → published
- 人工确认：必须点“Approve”才进入可发布状态
- 记录发布结果：video_url/video_id、发布时间、失败原因

### 3.6 Analytics + Feedback（数据与复盘）
- 采集指标（MVP可手动回填或半自动）：
  - views、likes、comments、shares（1h/6h/24h）
  - 可选：平均观看时长/完播率/商品点击/订单
- 规则引擎输出复盘建议：
  - 钩子弱（1h views 低、互动率低）
  - 结尾弱（互动高但点击/转化低）
  - 素材/模板疑似重复导致限流（同模板连续低表现）

## 4. 数据模型（MVP）
详见 `docs/DATA_MODEL.md`：
- products / product_assets
- script_templates / scripts
- renders
- accounts / publish_tasks
- published_videos（可合并进 publish_tasks）
- video_metrics / experiments

## 5. 技术栈（你已确认）
- API：TypeScript（NestJS）
- Console：TypeScript（Next.js）
- DB：Postgres
- Worker：Python（FFmpeg/MoviePy），Docker 化
- 本地开发：docker-compose

## 6. MVP里程碑（14天）
详见 `docs/ROADMAP.md`

## 7. 风控与合规（必须写进项目）
- 不提供/不合并任何“无人值守自动发帖/自动登录/绕过风控”功能
- 素材必须可证明权属：供应商授权、自制、或明确授权素材
- 二创必须具备“明显增量”：旁白/结构/镜头重构/字幕说明等
