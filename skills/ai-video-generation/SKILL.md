---
name: ai-video-generation
description: Generate AI videos with EzUGC catalog models (Sora 2, Veo, Kling, Runway, Seedance, and more). Use when the user wants text-to-video, image-to-video, a specific model, or "generate a video". Talking-head UGC ads use the ugc-generation skill instead.
---

# AI Video Generation

Generate videos with EzUGC's catalog of AI video models through MCP or the public API.

Requires a **paid** EzUGC account and an `ezk_live_` API key from https://app.ezugc.ai/dashboard/apps/mcp. If `whoami` fails, follow `ezugc-setup`. Never use app cookies or a browser session.

Never mention USD, provider cost, or credit-to-dollar rates. Quota is remaining videos this billing cycle (`get_usage`).

Install this skill:

```bash
npx skills add team-listnr/ez-mcp --skill ai-video-generation
```

## Quick start

```bash
# MCP client env
# EZUGC_API_KEY=ezk_live_…   npx -y @ezugc/mcp
```

1. Call `whoami`, then `get_usage`. Stop on auth failure.
2. Call `list_video_models`. Use a model with `available: true`. Do not invent an id.
3. Call `generate_video`. Prefer `wait: true`.
4. Report the real video URL from the job. Never invent a result.

Example:

```json
{
  "prompt": "Handheld vertical clip of a skincare bottle catching window light on a bathroom counter",
  "model": "google:3@3",
  "duration": 5,
  "aspectRatio": "9:16"
}
```

## Tool

Call **`generate_video`**. It queues `POST /api/public/video/jobs`.

| Arg | Required | Notes |
| --- | --- | --- |
| `prompt` | yes for text-to-video | Concrete visual brief. |
| `model` | no | Id from `list_video_models`. Omit to use the default. |
| `imageUrls` | no | Public https images for image-to-video. First image is the start frame. Required when the model is image-to-video only. |
| `duration` | no | Seconds; snapped to the model's supported options. |
| `aspectRatio` | no | `9:16`, `16:9`, or `1:1`. |
| `resolution` | no | Explicit size like `1280x720`. |
| `generateAudio` | no | Only if the chosen model supports audio. |
| brand refs | no | `brandId` / `brandSlug` / `useDefaultBrand`. |
| `wait` | no | Default true. |

## Picking a model

Call `list_video_models` and map the user's ask onto a live id. Common ids (verify they are still `available: true`):

- Sora 2 Pro: `openai:3@2` · Sora 2: `openai:3@1`
- Veo 3.1: `google:3@2` · Veo 3.1 Fast: `google:3@3`
- Kling 2.5 Turbo Pro: `klingai:6@1` · Runway Gen-4.5: `runway:1@2`
- Hailuo 2.3: `minimax:4@1` · Wan 2.5: `runware:201@1`

Respect each model's capabilities. Image-to-video models need `imageUrls`. Lip-sync, avatar, and OmniHuman models are not on this API.

## Format

- **Cinematic / product / b-roll / animation** — this skill (`generate_video`).
- **Spokesperson talking to camera (UGC ad)** — `ugc-generation` (`generate_ugc_video`).

If the brief is ambiguous, ask once, then generate.

## On-brand clips

If they want it for a saved brand: `list_brands` → `create_brand { websiteUrl }` if needed → pass `useDefaultBrand: true` (or `generate_brand_video` for a talking-head UGC ad).

## Hard rules

- Never proceed without a configured EzUGC API key.
- Never invent a job or video URL.
- Only request models returned as `available: true`.
- Do not dump provider names, task ids, or raw upstream errors to the user.
