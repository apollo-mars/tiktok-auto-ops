# Data Model (MVP)

Target: Vietnam (vi-VN), TikTok Shop, 3C small accessories.

This document defines the MVP database schema (Postgres) for the TikTok Auto Ops system.

## Conventions
- Primary keys: UUID (uuid_generate_v4())
- Timestamps: timestamptz
- Soft delete: deleted_at
- Money: integer in VND (price_vnd)
- Text search: store normalized columns when needed (e.g., name_vi_norm)

## Core entities
### 1. products
Represents one sellable item / SKU (or a product with variants if you extend later).

**Key fields**
- id (uuid, pk)
- sku (text, unique)
- name_vi (text)
- name_zh (text, nullable)
- category (text) e.g. "cap-sac", "cu-sac", "op-lung", "hub"
- price_min_vnd (int)
- price_max_vnd (int)
- core_benefits (jsonb) array of 1–3 strings
- proof_points (jsonb) array of strings (protocols, material, warranty)
- compatibility (jsonb) (models, protocols: PD/QC, connectors)
- offer (jsonb) (gift, free_shipping, warranty)
- landing_url (text) TikTok Shop product link
- compliance_flags (jsonb) (sensitive_claims, prohibited_words)
- status (text) draft|active|paused
- created_at, updated_at, deleted_at

### 2. product_assets
Stores media assets and rights/authorization metadata.

**Key fields**
- id (uuid, pk)
- product_id (uuid, fk -> products.id)
- asset_type (text) video|image|audio|subtitle|project
- source_type (text) self_made|supplier|licensed|remix_reference
- uri (text) (S3/minio path or local path)
- checksum (text) (sha256)
- duration_ms (int, nullable)
- width, height (int, nullable)
- tags (jsonb) (shot_type, pace, style, use_case)
- rights (jsonb) (license, authorization_doc_uri, expires_at)
- compliance_notes (text)
- created_at, updated_at, deleted_at

### 3. script_templates
Script template definitions.

- id (uuid)
- name (text)
- template_type (text) pain_solution|comparison|review|scenario|unboxing|faq
- language (text) vi|en|zh
- schema (jsonb) (slots: hook, pain, proof, cta)
- status (text) active|disabled
- created_at, updated_at

### 4. scripts
Instantiated scripts bound to a product.

- id (uuid)
- product_id (uuid)
- template_id (uuid)
- language (text) vi|en|zh
- content (jsonb) (structured segments + timing hints)
- hook_variant (text)
- cta_variant (text)
- created_at

### 5. renders
One render attempt (script + template + assets) producing a video.

- id (uuid)
- product_id (uuid)
- script_id (uuid)
- render_template (text) T1_compare|T2_pain|T3_unboxing
- config (jsonb) (assets chosen, subtitle style, voice, music)
- output_uri (text)
- status (text) queued|running|succeeded|failed
- error (text)
- created_at, updated_at

### 6. accounts
TikTok accounts managed in the ops console.

- id (uuid)
- handle (text)
- region (text) VN
- language (text) vi
- niche (text) 3c_accessories
- daily_cap (int) default 3
- publish_windows (jsonb) (e.g., [{start:"11:30", end:"13:30"},{start:"19:00",end:"22:30"}])
- status (text) active|paused
- created_at, updated_at

### 7. publish_tasks
Publishing queue with human confirmation gate.

- id (uuid)
- account_id (uuid)
- render_id (uuid)
- scheduled_at (timestamptz)
- state (text) pending_review|approved|published|skipped|failed
- approved_by (text, nullable)
- approved_at (timestamptz, nullable)
- published_video_url (text, nullable)
- published_video_id (text, nullable)
- failure_reason (text, nullable)
- created_at, updated_at

### 8. video_metrics
Metrics samples for published videos (1h/6h/24h).

- id (uuid)
- publish_task_id (uuid)
- sampled_at (timestamptz)
- views (int)
- likes (int)
- comments (int)
- shares (int)
- avg_watch_time_ms (int, nullable)
- completion_rate (numeric, nullable)
- product_clicks (int, nullable)
- orders (int, nullable)

### 9. experiments
A/B testing registry.

- id (uuid)
- dimension (text) hook|cta|template|caption|cover
- variant_a (text)
- variant_b (text)
- start_at, end_at
- success_metric (text) views_1h|ctr|orders

## Minimal indices (recommended)
- products(sku) unique
- product_assets(product_id)
- scripts(product_id, created_at desc)
- renders(status, created_at)
- publish_tasks(account_id, scheduled_at)
- video_metrics(publish_task_id, sampled_at)
