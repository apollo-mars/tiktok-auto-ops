Quickstart（docker compose）
How to contribute（指向 docs/CONTRIBUTING.md + Issue #1）

# TikTok Auto Ops (Vietnam) — Affiliate/Direct Supply Automation Stack

A community-built, compliance-first TikTok operations system for **Vietnam (vi-VN)** focused on **3C small digital accessories** (cables, chargers, cases, hubs, screen protectors, etc.).

Goal: maximize automation for **content production + scheduling + analytics**, while keeping a **human publish gate** (and optionally a human engagement gate) to reduce account risk.

## What this is
- A modular “content factory” + “ops console” to run 1–2 accounts at first, then scale to a multi-account matrix.
- Designed for **mixed editing / UGC-style remix (二创/混剪)**, but with strong emphasis on **rights-safe assets** and “transformative” editing.

## What this is NOT
- Not a bot for unattended posting, mass login automation, fake engagement, or bypassing platform security.
- Not a guarantee of sales or virality.

## MVP (14 days) scope
1. Product Hub: SKU + benefits + compatibility + offer + landing link
2. Script Engine: hook/pain/proof/CTA templates + variants (vi)
3. Asset Library: rights-safe assets, tagging, supplier proof
4. Video Composer: template-based auto rendering (hooks/subtitles/TTS)
5. Scheduler: publish task queue + human confirmation gate
6. Analytics: metrics sampling + auto post-mortem suggestions

## Repo structure (proposed)
- docs/
  - PRD.md
  - DATA_MODEL.md
  - ROADMAP.md
- services/
  - api/        # product/script/render/schedule APIs
  - worker/     # render + jobs
- apps/
  - console/    # ops console (web)
- packages/
  - core/       # shared types + rules

## How to contribute
See CONTRIBUTING.md and open an issue from the task list.

## Call for collaborators
We’re looking for:
- Backend: job queue, DB schema, REST APIs
- Video: template rendering pipeline (FFmpeg/MoviePy/CapCut templates)
- Frontend: ops console (Next.js/React)
- Data: metrics ingestion + A/B testing + dashboards

Open an issue or pick one labeled `good first issue`.

---

## Quick questions to decide (help wanted)
- Conversion path for MVP: TikTok Shop VN vs Shopee/Lazada vs COD landing page
- Preferred stack: TypeScript (Nest/Next) vs Python (FastAPI) for workers
