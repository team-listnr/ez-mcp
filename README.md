# ez-mcp

EzUGC plugins for **Claude Code** and **Cursor** — generate UGC/brand videos and static image ads with any model (Sora 2, Veo, Kling, Runway, Hailuo, Wan, LTX, Seedance, and more), grounded in your brand.

This public repo is the plugin marketplace for both clients:

| Client | Manifest | MCP transport |
| --- | --- | --- |
| Claude Code | `.claude-plugin/` | Local `npx -y @ezugc/mcp` + `EZUGC_API_KEY` |
| Cursor | `.cursor-plugin/` | Remote `https://api.ezugc.ai/mcp` + OAuth |

Paid EzUGC plan required. Generate a key at [app.ezugc.ai/dashboard/apps/mcp](https://app.ezugc.ai/dashboard/apps/mcp).

## Cursor

One-click install (works before the Cursor Marketplace listing is approved):

[Add EzUGC to Cursor](https://cursor.com/install-mcp?name=ezugc&config=eyJ1cmwiOiJodHRwczovL2FwaS5lenVnYy5haS9tY3AifQ==)

Or add this to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "ezugc": {
      "url": "https://api.ezugc.ai/mcp"
    }
  }
}
```

Cursor will prompt you to authorize. Paste your `ezk_live_` key on the consent screen.

To submit / update the Cursor Marketplace listing, this repo is the plugin source: [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish).

Local test:

```bash
ln -s /path/to/ez-mcp/plugins/ez ~/.cursor/plugins/local/ezugc
```

Reload Cursor and confirm the EzUGC MCP server plus the brand skills appear in Customize.

## Claude Code

1. Export your paid EzUGC API key in the shell you launch Claude Code from:

   ```bash
   export EZUGC_API_KEY="ezk_live_..."
   ```

2. Add the EzUGC marketplace:

   ```
   /plugin marketplace add team-listnr/ez-mcp
   ```

3. Install the `ez` plugin:

   ```
   /plugin install ez@ezugc
   ```

Claude Code starts `@ezugc/mcp` via npx and reads `EZUGC_API_KEY` from the environment. Node.js 18+ required for this path.

## What you get

- **MCP server** (`ezugc`): `whoami`, `get_usage`, video/image generation, brands, saved skills, jobs, and Super Agent tools.
- **Skills:** `brand-video` (pick a model, ingest a website, generate on-brand video/image ads) and `brand-assets` (inspect/update the brand book and reuse assets).

## Usage

- "Check my EzUGC account with whoami, then list the video models."
- "Make a 15s launch teaser with Sora 2."
- "Create a video for my brand from acme.com."
- "Generate a static image ad for my brand, 4 variations, 1:1."

## Notes

- A **paid** `ezk_live_` key is required. Unpaid/invalid keys stop the tools. Subscribe at [app.ezugc.ai](https://app.ezugc.ai/) — do not retry blindly.
- This plugin package is markdown + config only. The hosted MCP and API stay on `api.ezugc.ai`.

Learn more at [www.ezugc.ai/mcp](https://www.ezugc.ai/mcp).
