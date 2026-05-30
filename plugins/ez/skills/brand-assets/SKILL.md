---
name: brand-assets
description: Work with a saved EzUGC brand's assets and brand book. Look up the brand's logo, colors, and product images, review or update its brand_md (brand DNA), and use those assets to generate on-brand image ads and videos through the EzUGC MCP server. Use when the user wants to inspect, edit, or generate from their saved brand identity.
---

# EzUGC Brand Assets Skill

Use this skill when the user wants to:

- **Look up a saved brand**: its logo, color palette, product images, tone, and value props.
- **Review or update the brand book** (`brandMd` / `brand_md`) — the Markdown brand DNA that grounds generations.
- **Use brand assets** (uploaded assets + previously generated creatives) as reference images for on-brand image ads or videos.

This skill is **MCP-first**: every action below is an EzUGC MCP tool call. There is no CLI fallback.

## Required Preconditions

- An EzUGC account with an **active paid subscription** — the API will not issue a key without one.
- A **paid** EzUGC API key (prefix `ezk_live_`) configured via the `EZUGC_API_KEY` environment variable.
- The plugin wires this through `plugins/ez/.mcp.json`, so export it before launching Claude Code:
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

## Find the Brand

1. `list_brands` — lists saved brands (id, slug, website, status, brand name, logos).
2. If none exists, create one from the user's website: `create_brand { websiteUrl }`, then poll `get_brand { brandId }` until `status` is `ready`.
3. To reference a brand you can pass `brandId`, `brandSlug`, or `useDefaultBrand: true` (the most recently ready brand).

## Inspect Brand Identity & Assets

- `get_brand { brandId }` returns the brand profile: `brandMd` and `brandMdVersion`, `toneOfVoice`, `valueProps`, `colorPalette`, `logoUrls`, and `productImageUrls`.
- `get_brand_assets { brandId | brandSlug | useDefaultBrand }` returns the brand's stored assets:
  - `source: "upload"` — user-uploaded brand assets.
  - `source: "generated"` — previously generated brand creatives (with the `prompt` used).
  - Each asset includes a temporary signed `url`. Treat these URLs as short-lived; re-fetch before reuse.

## Review / Update the Brand Book (`brand_md`)

1. Read the current brand book with `get_brand` (the `brandMd` field).
2. Edit it as Markdown — keep the brand's real identity (name, tone, value props, audience, do/don't guidance).
3. Save it with `update_brand_md { brandMd, toneOfVoice?, brandId | brandSlug | useDefaultBrand }`.
   - `brandMd` replaces the existing brand book; send the full updated Markdown, not a diff.
   - Editing `brandMd` automatically bumps `brandMdVersion`.
   - `toneOfVoice` is optional and updated alongside.
4. Confirm by reading back the returned `profile.brandMd` / `brandMdVersion`.

## Generate On-Brand Creative From Assets

1. Get reference URLs from `get_brand_assets` (and/or `logoUrls` / `productImageUrls` from `get_brand`).
2. Image ad: `generate_image_ad { prompt, referenceImageUrls: [<asset urls>], aspectRatio?, count? }`, or `generate_brand_image_ad { prompt }` to auto-ground in the saved brand.
3. Video: `generate_video { prompt, model?, imageUrls: [<asset urls>], useDefaultBrand?: true }`, or `generate_brand_video { prompt }` for a brand-grounded UGC ad.
4. Prefer waiting for completion (`wait: true`). For long jobs, return the job id and poll `get_job { jobId }`. Report the real result URL — never invent one.

## Hard Rules

- Never proceed without a configured **paid** `ezk_live_` EzUGC API key in `EZUGC_API_KEY`.
- If any tool returns an auth/subscription error or no key is configured, STOP and tell the user to sign up & subscribe at https://app.ezugc.ai/, create an API key, and set `EZUGC_API_KEY`. Do NOT retry the call blindly.
- When updating `brandMd`, send the complete Markdown document (it replaces the stored value).
- Signed asset URLs expire — re-fetch with `get_brand_assets` before reusing.
- Never invent a brand profile, asset URL, or job result. Always report the actual API response.
- Never use app cookies, Supabase tokens, or browser sessions for public API work.
