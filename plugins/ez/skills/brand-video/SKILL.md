---
name: brand-video
description: Generate on-brand EzUGC videos and image ads with any model (Sora 2, Veo, Kling, Runway, Hailuo, Wan, LTX, Seedance, and more) and create assets "for my brand" by ingesting the user's website. Use when the user wants a video or static ad with a specific model, or an ad grounded in their brand identity. Works through the EzUGC MCP server.
---

# EzUGC Brand Video Skill

Use this skill when the user wants to:

- Generate a video with a **specific model** (e.g. "make this with Sora 2", Veo 3.1, Kling, Runway).
- Create a video or image ad **"for my brand"** that matches their logo, colors, tone, and products.
- Generate a **static image ad**.
- Save / inspect a brand built from their website.

This skill is **MCP-first**: every action below is an EzUGC MCP tool call. There is no CLI fallback.

## Required Preconditions

- An EzUGC account with an **active paid subscription** — the API will not issue a key without one.
- A **paid** EzUGC API key (prefix `ezk_live_`) configured via the `EZUGC_API_KEY` environment variable.
- The plugin wires this through `plugins/ez/.mcp.json`, so export it in your shell before launching Claude Code:
  - `export EZUGC_API_KEY="ezk_live_..."`
- **If no key is configured**, or `whoami`/any tool reports an auth/subscription error (e.g. the key is unpaid/invalid), **STOP** and tell the user to:
  1. Sign up / log in and subscribe at https://app.ezugc.ai/
  2. Create an API key in account settings.
  3. Set `EZUGC_API_KEY=ezk_live_...` in their environment.
- Never fall back to browser/app cookies, Supabase tokens, or the dashboard session. Do NOT retry blindly — direct the user to https://app.ezugc.ai/.

## Verify Auth First

1. Call `whoami` to confirm the key, account, scopes, and `paymentStatus`.
2. Call `get_usage` to check remaining quota.
3. If either fails (401/402) or `paymentStatus` is not `paid`, stop and report it. Do not attempt generation.

## Picking a Model

1. Call `list_video_models` to see available models, capabilities, durations, and ids.
2. Map the user's intent to a model id. The catalog returns ids such as Sora 2 Pro / Sora 2, Veo 3.1 / Veo 3.1 Fast, Kling 2.5 Turbo Pro, Runway Gen-4.5, Hailuo 2.3, Wan 2.5, LTX-2, and Seedance. **Always read the live list rather than hardcoding ids** — the catalog changes.
3. Respect each model's capabilities: image-to-video models require image inputs (`imageUrls`); lip-sync / avatar / OmniHuman models are reported as `available: false` and must use the dashboard.
4. Only request models returned as `available: true`.
5. Duration is snapped to the model's supported options automatically.

## Generating a Video

- `generate_video { prompt, model?, imageUrls?, duration?, aspectRatio?, resolution?, generateAudio? }`
  - Omit `model` to use the default model.
  - Pass `imageUrls` (public https) for image-to-video; the first image is the start frame.
  - Defaults to waiting for completion. Set `wait: false` to get a job id and poll `get_job { jobId }`.

## UGC Ad Videos

- `generate_ugc_video { prompt, script?, productImageUrls?, platform?, duration?, aspectRatio? }`
  - Produces a full UGC-style ad (script + avatar + voiceover + captions) from a brief.
  - Omit `script` to auto-generate it.

## Brand Videos ("create a video for my brand")

1. Check for a saved brand with `list_brands`.
2. If none exists, create one from the user's website:
   - `create_brand { websiteUrl }` — EzUGC crawls the site and extracts brand DNA (logo, colors, tone, value props).
   - Ingestion runs in the background. Poll `get_brand { brandId }` until `status` is `ready`.
3. Generate the brand video:
   - `generate_brand_video { prompt }` uses the default saved brand automatically — best for "create a video for my brand".
   - Or `generate_video { prompt, model, useDefaultBrand: true }` to pick a specific model while grounding in the brand.
4. The brand's name, website, tone, colors, value props, and product/logo images are injected automatically.

## Image Ads ("make me a static ad")

1. Call `list_image_models` to see image-ad models, reference-image support, and aspect ratios. Read the live list for ids.
2. Generate 1-4 images:
   - `generate_image_ad { prompt, model?, referenceImageUrls?, aspectRatio?, count? }` — pass `referenceImageUrls` (product, logo, prior creatives) to ground the output.
   - `generate_brand_image_ad { prompt }` for an ad grounded in the saved brand.
3. Results land in `result_payload.images[]` (each has a `url`). Image jobs charge `staticAdGenerations` usage (one per image).

## Saved Skills (reusable prompts)

- `list_agent_skills` returns the read-only agent capability catalog.
- Saved skills are reusable parameterized prompt templates (use `{{variable}}` tokens) with a `kind` of `video`, `ugc`, or `image`:
  - `list_skills`, `create_skill { name, kind, promptTemplate, defaultParams? }`, `get_skill { skillId }`, `run_skill { skillId, variables?, overrides? }`.
  - Running a skill dispatches a job of its kind (`agent` kind is not runnable here).

## Workflow

1. Verify auth: `whoami` / `get_usage`. Stop on failure or unpaid key.
2. Pick the model (see above) unless the user is fine with the default.
3. Resolve or create the brand if the request is brand-specific.
4. Generate. Prefer waiting for completion (`wait: true`). For long jobs, return the job id and poll `get_job { jobId }`.
5. Report the real result (video/image URL) — never invent one.

## Hard Rules

- Never proceed without a configured **paid** `ezk_live_` EzUGC API key in `EZUGC_API_KEY`.
- If any tool returns an auth/subscription error (e.g. `PUBLIC_API_KEY_REQUIRED`, `PUBLIC_API_KEY_INVALID`, `PUBLIC_API_SUBSCRIPTION_REQUIRED`) or no key is configured, STOP and tell the user to sign up & subscribe at https://app.ezugc.ai/, create an API key, and set `EZUGC_API_KEY`. Do NOT retry the call blindly.
- Never invent a job/video/image result. Always report the actual API response.
- Never use app cookies, Supabase tokens, or browser sessions for public API work.
- Only request models returned as `available: true` by `list_video_models` / `list_image_models`.
