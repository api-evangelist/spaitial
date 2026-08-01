---
name: spaitial-export-mesh
description: Export a collision/reconstructed mesh from a completed SpAItial world and download it when ready.
api: SpAItial Developer API
base_url: https://api.spaitial.ai
operations:
  - V1Worlds_createExport # POST /v1/worlds/requests/{requestId}/exports/{type}
  - V1Worlds_getExport    # GET  /v1/worlds/requests/{requestId}/exports/{type}
  - V1Worlds_listExports  # GET  /v1/worlds/requests/{requestId}/exports
scopes: [worlds:write, worlds:read]
---

# Export a mesh from a completed world

Derive a mesh artifact from a completed world and fetch the file.

## Steps
1. **Start (or retrieve) the export** — `V1Worlds_createExport`
   (`POST /v1/worlds/requests/{requestId}/exports/{type}`) with `type` = `mesh` (full-resolution `.ply`)
   or `mesh-simplified` (real-time optimized). Requesting either starts the shared mesh pipeline.
   Returns `202` (started) or `200` (already ready). Requires the world to be `COMPLETED`
   (else `409 RESOURCE_NOT_READY`).
2. **Poll** — `V1Worlds_getExport` (`GET /.../exports/{type}`). `status` is `PROCESSING` then `READY`.
   Mesh exports can take several minutes.
3. **Download** — when `READY`, the response includes a stable proxy `download_url`.
   Call `V1Worlds_getExport` with `?download=1` on the same endpoint to `302` to a short-lived signed file URL.
4. Optionally list all exports for the request — `V1Worlds_listExports` (`GET /.../exports`).

## Rules
- Store the stable API `download_url`; re-hit it for a fresh redirect (signed URLs last ~5 min).
- Exports are keyed by `type`, so new export types can appear without changing the route shape.
- Export webhooks (`world.export.completed` / `world.export.failed`) fire if a `webhook.url` was set on the world.
