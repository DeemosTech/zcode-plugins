---
name: hyper3d-rodin
description: Generate Hyper3D Rodin Gen-2.5 models from text or images, split completed models with BANG, track progress, and retrieve result pages or requested model files.
---

# Hyper3D Rodin

Use the tools discovered from the `hyper3d-rodin` MCP server in ZCode. Its server
namespace is `plugin:hyper3d-rodin:hyper3d-rodin`; match the tool names below to
the actual session tools instead of constructing a host-specific tool prefix.
Sign in to Hyper3D through ZCode's MCP OAuth flow; do not ask for or handle a
Hyper3D session token. If authorization or required scopes are missing, use the
host's reconnect/login flow. If that flow is unavailable or fails, report the
connection problem rather than asking for credentials in chat.
Billing uses the personal or group workspace selected during OAuth; reconnect
to change that selection. Generation and BANG consume that workspace's credits.
Explain this before the first paid submission. Submit only for a user-requested
generation or split; setup, connection tests, and status checks do not authorize
a new paid task. Do not claim to know the balance or exact price from this MCP.

## Generate, monitor, and retrieve

1. Call `rodin_generate` once with the user's prompt, reference images, or both.
   `prompt`, when supplied, must be non-empty and at most 1,024 characters.
   For image-only generation omit `prompt` instead of sending an empty string.
2. Save `generation_id` and `display_url`. Do not present the result-page link
   until the generation completes.
3. For a single progress check, call `rodin_get_status` with `generation_id`.
   To wait for completion, call `rodin_wait` with `generation_id` and
   `timeout_seconds` (integer 1–45, backend default 30). Keep each wait within
   the host's request timeout.
4. Repeat bounded waits while status is `queued` or `processing`, keeping the
   user informed. Report `stage.name` and `stage.current`/`stage.total` as
   stages, never as a percentage or time estimate. `current: 0` means no stage
   has started; do not invent progress when `total` is 0.
5. `timed_out: true` means the wait ended, not that generation failed. If status
   is `failed`, stop waiting and report the returned `error` when available;
   do not fetch results or automatically start another generation.
6. On `completed`, call `rodin_get_result`. Present its permanent `display_url`
   as the user-facing result link. Never expose or embed temporary `files[].url`
   in the response. Only use those signed URLs to download files when the user
   explicitly requests a download; return the resulting local file artifacts.
   Select files by `role` (`model`, `preview`, `texture`, `sidecar`) and include
   required textures/sidecars when downloading a multi-file model format.

## Reference images

In ZCode, use actual local file paths supplied by the user or available from
their attachments with the PUT workflow below. Never fabricate paths or upload
IDs. `rodin_import_images` is only for ChatGPT Chat; do not call it in ZCode even
if the server includes it in tool discovery.

1. Inspect the actual files. Each must be a supported, decodable image, at most
   20 MiB (20,971,520 bytes). Use the tool schema's `mime_type` enum, such as
   `image/png`, `image/jpeg`, or `image/webp`; MIME type must match the bytes.
2. Call `rodin_create_uploads` with `files`, an ordered array of 1–5 objects:
   `filename` (basename only, 1–128 characters, no path separators or control
   characters), `mime_type`, and `size_bytes` (exact positive integer byte size).
3. For every returned `uploads[]` entry, upload the matching file using the
   returned `method`, `headers`, and `upload_url` before `expires_at`. Preserve
   the exact content type; confirm every PUT succeeds before generating.
4. Pass the ordered, unique `uploads[].upload_id` values as
   `reference_upload_ids`, with an optional prompt, to `rodin_generate` once.
   Follow the monitoring and result workflow above.

Upload records expire after two hours and are consumed by a successful
submission. Do not reuse upload IDs for a second generation; prepare fresh
uploads for a new user-requested generation. Never reuse an uncertain submission
as a reason to upload and generate again automatically.

If PUT is blocked and the host supports a network permission flow, request
access to the returned upload hostname and retry PUT after access is granted.
If uploading remains unavailable, explain in the user's language that this
model environment cannot upload images and they can upload and generate at
https://hyper3d.ai. Do not silently omit references or switch to text-only.

## Rodin settings

Use the live tool schema for available fields; do not pass website or CLI
settings that the MCP tool does not expose.

- `mesh_mode`: `Raw` (default) or `Quad`.
- `tier`: `Gen-2.5-Medium` (default), `Gen-2.5-High`, or `Gen-2.5-Extreme-Low`.
- `texture_delight`: boolean, default `false`. Recommend enabling it for
  reference images with highly reflective surfaces; respect the user's choice.
- `quality_override`: optional integer target polygon count; 500–1,000,000 for
  `Raw` or 1,000–50,000 for `Quad`. When omitted the backend uses 500,000 for
  `Raw` or 18,000 for `Quad`, independently of the selected tier.
- `geometry_file_format`: `glb` (default), `usdz`, `fbx`, `obj`, or `stl`.

## BANG part splitting

Use a completed Rodin generation belonging to the authorized user. Pass the
`generation_id` returned by `rodin_generate` directly as `asset_id`; do not pass
a file URL or upload ID. Arbitrary local model uploads are not exposed by the
current MCP tool set.

Call `rodin_generate_bang` once with `asset_id` and requested options:

- `instruction`: optional text naming the parts the user wants separated.
  Omit it unless the user specifies parts; omission uses the automatic planner.
- `strength`: integer 1–12, default 5; a soft component-count target, not an
  exact guarantee.
- `explode_strength`: number at least 0, default 1; split-plan guidance strength.
- `geometry_file_format`: the same five formats as Rodin; default `glb`.
- `resolution`: `Basic` (default) or `High`. `High` requires account entitlement.
- `seed`: optional integer 0–65,535; `escore` and `reference_scale`: optional
  numbers. If omitted these three settings inherit from the source task;
  leave them unset unless requested.

Save the new `generation_id` and monitor it using `rodin_wait` or
`rodin_get_status`, then retrieve its result with `rodin_get_result`.

## Billing, privacy, and recovery

- `rodin_generate` and `rodin_generate_bang` consume credits and are not
  idempotent. Never automatically retry either tool after a timeout or
  connection error. If an ID is known, check that task; otherwise ask the user
  to inspect Hyper3D Mine before deciding whether to submit another generation.
- On insufficient balance or unavailable entitlement, report the error; do not
  silently change the billing workspace or downgrade requested settings.
- The backend makes group-workspace and paid-subscriber generations private.
  Do not promise that a free personal-account generation is private; there is
  no MCP privacy parameter.
- Status/result access is limited to the authorized user's generations. A group
  billing grant does not imply access to other members' models.
- Keep presigned upload/download URLs out of user-facing responses and shared
  logs. For user-facing result links, use only the completed task's `display_url`.
- The current MCP exposes generation/upload/status/result tools, not account
  balance or generation-history listing tools. Do not invent those calls.
- Do not import models into Blender, Unreal, or another DCC. A DCC-specific
  plugin or skill owns that workflow.
