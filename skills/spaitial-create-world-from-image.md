---
name: spaitial-create-world-from-image
description: Upload an image to SpAItial and generate a 3D Gaussian-splat world from it, then download the splat.
api: SpAItial Developer API
base_url: https://api.spaitial.ai
operations:
  - V1Files_uploadFile        # POST /v1/files
  - V1Worlds_createJob        # POST /v1/worlds
  - V1Worlds_getJobStatus     # GET  /v1/worlds/requests/{requestId}/status
  - V1Worlds_getJobResult     # GET  /v1/worlds/requests/{requestId}
  - V1Worlds_getSplatDownload # GET  /v1/worlds/requests/{requestId}/splat
scopes: [files:create, worlds:create, worlds:read]
---

# Create a 3D world from an image

Upload a photo (or 360 panorama), generate a world from it, and download the splat.

## Steps
1. **Upload** — `V1Files_uploadFile` (`POST /v1/files`, `multipart/form-data`, field `file`).
   Accepts JPEG/PNG/WebP/GIF. Returns a `file_id` (24-hour TTL, owner-scoped).
   (Alternatively skip upload and pass `input.type:"url"` with an HTTPS `image_url`, or `type:"base64"` ≤25MB.)
2. **Submit** — `V1Worlds_createJob` (`POST /v1/worlds`) with
   `{"input":{"type":"file_id","file_id":"<file_id>"},"title":"My world"}`.
   For an equirectangular 360 panorama, add `"is_pano":true` to the input.
   Send an `Idempotency-Key` header. Returns `202` + `request_id`.
3. **Poll** — `V1Worlds_getJobStatus` until `COMPLETED`.
4. **Fetch result** — `V1Worlds_getJobResult`; read `world.splat_url` / `world.viewer_url`.
5. **Download** — `V1Worlds_getSplatDownload` (`302` → signed URL; follow with `-L`).

## Rules
- Optional suitability check: set `validation.skip:false` (advisory) or add `error_on_fail:true` (strict → `422 VALIDATION_FAILED` with `details.validation.issues`).
- Moderation always runs; unsafe inputs return `403 MODERATION_REJECTED` before any credit is spent.
- Using a `file_id` marks it `consumed`; expired/unknown ids return `404 FILE_NOT_FOUND` / `FILE_EXPIRED`.
- Reads and downloads are not charged; only generation consumes credits.
