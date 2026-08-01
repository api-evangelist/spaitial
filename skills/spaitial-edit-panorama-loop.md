---
name: spaitial-edit-panorama-loop
description: Iteratively edit a world's panorama with prompts on SpAItial, inspect each result, then create a new world from the final panorama.
api: SpAItial Developer API
base_url: https://api.spaitial.ai
operations:
  - V1Panoramas_editPanorama     # POST /v1/panoramas/edit
  - V1Panoramas_downloadPanorama # GET  /v1/panoramas/{panoramaId}/download
  - V1Worlds_createJob           # POST /v1/worlds  (input.type = panorama_id)
  - V1Worlds_getJobStatus        # GET  /v1/worlds/requests/{requestId}/status
scopes: [worlds:create, worlds:read]
---

# Edit a panorama, iterate, then generate a world

The app-style "edit the panorama, inspect it, iterate, then generate a new world" flow.

## Steps
1. **Edit** — `V1Panoramas_editPanorama` (`POST /v1/panoramas/edit`) with a `source` and a `prompt`.
   `source` is one of:
   - `{"type":"request_id","request_id":"req_..."}` — an API-created completed world request you own.
   - `{"type":"world_id","world_id":"<uuid>"}` — an API-created completed world you own.
   - `{"type":"panorama_id","panorama_id":"pano_..."}` — a previous edit artifact (to iterate).
   Optionally pass up to 3 reference `images` (`url` / `base64` / `file_id`).
   Panorama edits are **synchronous**; the response includes a `panorama_id` (`pano_...`, 24-hour TTL).
   Send an `Idempotency-Key` header.
2. **Inspect** — `V1Panoramas_downloadPanorama` (`GET /v1/panoramas/{panoramaId}/download`), `302` → signed URL.
3. **Iterate** — call `V1Panoramas_editPanorama` again with `source.type:"panorama_id"` set to the returned `pano_...` and a new prompt. Repeat until satisfied.
4. **Create the world** — `V1Worlds_createJob` (`POST /v1/worlds`) with
   `{"input":{"type":"panorama_id","panorama_id":"<final pano_...>"},"title":"Edited world"}`, then poll `V1Worlds_getJobStatus`.

## Rules
- There is intentionally no aspect-ratio field — SpAItial preserves the panorama format so the result stays valid for world generation.
- Expired panoramas return `410 PANORAMA_EXPIRED`; a transient upstream failure returns `502 EDIT_FAILED` (retry).
- Editing spends `worlds:create` scope and credits; inspection/download does not.
