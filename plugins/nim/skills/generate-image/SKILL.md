---
name: nim-generate-image
description: Use when a user asks Claude to create or edit an image with Nim. Guides model discovery, input selection, generation, polling, and result links through the Nim MCP.
---

# Nim Image Generation

Use this skill to generate or edit images with Nim through MCP.

## Workflow

1. Call `models_explore` with `action: "recommend"` or `action: "search"` and `type: "image"` when the model is not already chosen.
2. Call `models_explore` with `action: "get"` for the final `model_id`.
3. Follow the returned `generationContract` exactly. Do not pass unsupported params.
4. Call `generate_image`.
5. Poll `get_generation_status` until the generation reaches `finished`, `failed`, or `cancelled` when the client does not render the Nim widget.
6. Use `generationUrl` as the Nim link when available. Do not invent Nim URLs.

## Inputs

- `prompt` is required and should contain the full image intent.
- `model_id` is required and must come from `models_explore`.
- `model_name` is optional, but include it when available for clearer user-facing status.
- `requestedAspectRatio`, `resolution`, `fileInputs`, and `seed` are optional only when the selected model contract allows them.
- For image-edit or image-reference models, upload/import reference media first and pass the resulting file paths as `fileInputs`.
- Use `batchSize` for variations only when the user asks for multiple options.

## Output Handling

- Prefer the Nim generation widget when the client supports MCP Apps/resources.
- Do not describe queued generations as completed.
- If a generation fails, show the returned `error`/`errorCode` when present and avoid retrying with changed parameters unless the user asks.
