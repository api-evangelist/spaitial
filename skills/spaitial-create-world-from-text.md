---
name: spaitial-create-world-from-text
description: Generate an explorable 3D Gaussian-splat world from a text prompt with the SpAItial Developer API, then download the splat.
api: SpAItial Developer API
base_url: https://api.spaitial.ai
operations:
  - V1Worlds_createJob        # POST /v1/worlds
  - V1Worlds_getJobStatus     # GET  /v1/worlds/requests/{requestId}/status
  - V1Worlds_getJobResult     # GET  /v1/worlds/requests/{requestId}
  - V1Worlds_getSplatDownload # GET  /v1/worlds/requests/{requestId}/splat
scopes: [worlds:create, worlds:read]
---

# Create a 3D world from text

Generate a world from a natural-language prompt and download the resulting Gaussian splat.

## Prerequisites
- A SpAItial API key (`spt_live_...`) from https://developers.spaitial.ai, sent as `Authorization: Bearer <key>`.
- Plan credit on the account (Echo 2 - Standard costs 160 credits; free daily credit cannot pay for API usage).

## Steps
1. **Submit** the job — `V1Worlds_createJob` (`POST /v1/worlds`) with
   `{"input":{"type":"text","prompt":"a cozy sunlit reading nook"},"title":"Reading nook"}`.
   Send an `Idempotency-Key` header (a UUID) so a network retry does not create a duplicate job.
   Returns `202` with `request_id` and `status: "PENDING"`.
2. **Poll** — `V1Worlds_getJobStatus` (`GET /v1/worlds/requests/{requestId}/status`) every 5-10s.
   Generation takes ~5-10 minutes. Stop when `status` is `COMPLETED`, `FAILED`, or `CANCELLED`.
3. **Fetch result** — `V1Worlds_getJobResult` (`GET /v1/worlds/requests/{requestId}`) once `COMPLETED`.
   The `world` object contains `splat_url`, `panorama_url`, `thumbnail_url`, and `viewer_url`.
4. **Download the splat** — `V1Worlds_getSplatDownload` (`GET /v1/worlds/requests/{requestId}/splat`).
   It returns `302` to a ~5-minute signed URL; follow the redirect (curl `-L`).

## Rules
- Branch on the stable `error.code`, not the message (see errors/spaitial-problem-types.yml).
- `402 INSUFFICIENT_CREDITS` → purchase credits or upgrade the plan; do not retry blindly.
- `429 RATE_LIMIT_EXCEEDED` → back off using `Retry-After` and `X-RateLimit-*`. World-create bucket is 10/min.
- Prefer a webhook (`webhook.url` on the create call) over tight polling for production.
