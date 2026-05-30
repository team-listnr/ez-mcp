# ez-mcp

EzUGC for Claude Code — generate UGC/brand videos and static image ads with **any model** (Sora 2, Veo, Kling, Runway, Hailuo, Wan, LTX, Seedance, and more) directly from Claude, grounded in your brand.

This repo is a **Claude Code plugin marketplace** (`ezugc`) that ships the `ez` plugin. The plugin launches the published [`@ezugc/mcp`](https://www.npmjs.com/package/@ezugc/mcp) MCP server via `npx`.

## Requirements

- A **paid** EzUGC account and API key (prefix `ezk_live_`). Get one at [www.ezugc.ai](https://www.ezugc.ai).
- Node.js 18+ (so `npx` can run the MCP server).

## Install

1. Export your paid EzUGC API key in the shell you launch Claude Code from:

   ```bash
   export EZUGC_API_KEY="ezk_live_..."
   ```

2. Add the EzUGC marketplace in Claude Code:

   ```
   /plugin marketplace add team-listnr/ez-mcp
   ```

3. Install the `ez` plugin from the `ezugc` marketplace:

   ```
   /plugin install ez@ezugc
   ```

Claude Code will start the `ezugc` MCP server (`npx -y @ezugc/mcp`) and read your `EZUGC_API_KEY` from the environment.

## What you get

- **MCP server** (`ezugc`): tools for `whoami`, `get_usage`, `list_video_models`, `generate_video`, `generate_ugc_video`, `generate_brand_video`, `list_brands`, `create_brand`, `get_brand`, `list_image_models`, `generate_image_ad`, `generate_brand_image_ad`, saved skills, and `get_job`.
- **Skill** `/ez:brand-video`: an MCP-first guide for generating on-brand videos and image ads with any model.

## Usage

Once installed, just ask Claude in natural language, e.g.:

- "Make a 15s launch teaser with Sora 2."
- "Create a video for my brand from acme.com."
- "Generate a static image ad for my brand, 4 variations, 1:1."

The `/ez:brand-video` skill auto-invokes for these requests.

## Notes

- A **paid** `ezk_live_` key is required. Unpaid/invalid keys return 401/402 and the tools will stop.
- The plugin contains no version field in `plugin.json`, so every commit to this repo acts as a release.

Learn more at [www.ezugc.ai](https://www.ezugc.ai).
