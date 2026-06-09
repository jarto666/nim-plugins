---
name: nim-generate-video
description: Use when a user asks Claude to create a video or image-to-video result with Nim. Guides model discovery, duration/resolution inputs, generation, polling, and result links through the Nim MCP.
---

# Nim Video Generation

Use this skill to generate videos with Nim through MCP.

## Workflow

1. Call `models_explore` with `action: "recommend"` or `action: "search"` and `type: "video"` when the model is not already chosen.
2. Call `models_explore` with `action: "get"` for the final `model_id`.
3. Follow the returned `generationContract` exactly. Do not pass unsupported params.
4. Call `generate_video`.
5. Poll `get_generation_status` until the generation reaches `finished`, `failed`, or `cancelled` when the client does not render the Nim widget.
6. Use `generationUrl` as the Nim link when available. Do not invent Nim URLs.

## Inputs

- `prompt` is required and should contain the full motion/video intent.
- `model_id` is required and must come from `models_explore`.
- `model_name` is optional, but include it when available for clearer user-facing status.
- `requestedAspectRatio`, `resolution`, `mediaLength`, `fileInputs`, `fps`, `keepSound`, and `seed` are optional only when the selected model contract allows them.
- `mediaLength` is an input duration in milliseconds, for example `5000` or `8000`.
- For image-to-video models, upload/import the reference image first and pass the resulting file path as `fileInputs`.
- Use `batchSize` for variations only when the user asks for multiple options.

## Output Handling

- Prefer the Nim generation widget when the client supports MCP Apps/resources.
- Do not describe queued generations as completed.
- If a generation fails, show the returned `error`/`errorCode` when present and avoid retrying with changed parameters unless the user asks.
