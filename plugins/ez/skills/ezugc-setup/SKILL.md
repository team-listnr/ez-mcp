---
name: ezugc-setup
description: Connect an agent to a paid EzUGC account with an API key. Use when the user wants to set up EzUGC, EZUGC_API_KEY is missing, whoami fails, or generation tools return an auth or subscription error.
---

# EzUGC Setup

Walk the user through connecting EzUGC. Stop as soon as auth works — then hand off to `ai-video-generation` or `ugc-generation`.

Install this skill:

```bash
npx skills add team-listnr/ez-mcp --skill ezugc-setup
```

## Hard rules

- Never use app cookies, Supabase tokens, or a browser session. Auth is an API key only.
- Never invent a successful `whoami` result. Call the tool and report what it returned.
- Never mention USD, provider cost, margins, or credit-to-dollar rates. Quota is remaining videos this billing cycle.
- If the key is missing or invalid, stop. Do not retry generation tools. Do not dump stack traces — explain the next step in plain language.

## 1. Confirm they have a paid EzUGC account

The public API issues keys to **paid** plans only. Free accounts cannot generate a key.

If they are not subscribed yet, send them to https://app.ezugc.ai/ to sign up and start a paid plan, then continue.

## 2. Create an API key

Send them to the dashboard key page:

https://app.ezugc.ai/dashboard/apps/mcp

They should create a key that starts with `ezk_live_`. Tell them to copy it once — they will not see the full key again.

## 3. Set `EZUGC_API_KEY`

The MCP server reads `EZUGC_API_KEY` from the environment.

- Claude Code: set `EZUGC_API_KEY` in the environment (shell profile, or the MCP / plugin env for this server), then restart Claude Code or reload plugins so the server picks it up.
- Other MCP clients: put the key in the server `env` block, for example:

```json
{
  "mcpServers": {
    "ezugc": {
      "command": "npx",
      "args": ["-y", "@ezugc/mcp"],
      "env": { "EZUGC_API_KEY": "ezk_live_…" }
    }
  }
}
```

Remote Streamable HTTP MCP is `https://api.ezugc.ai/mcp`. Send the same key as `Authorization: Bearer ezk_live_...` or `x-api-key`.

Never paste the full key back into the chat after they have set it. Confirm only that a key is configured.

## 4. Verify with `whoami`

Call `whoami`.

- Success: greet them with the account identity and `paymentStatus`. If `paymentStatus` is not `paid`, stop and send them to https://app.ezugc.ai/ to resubscribe.
- Missing key: tell them no EzUGC API key was found, point at https://app.ezugc.ai/dashboard/apps/mcp, and stop.
- Invalid / unauthorized key: tell them the key was rejected (wrong, revoked, or no active paid plan). Point at the same dashboard URL. Stop. Do not retry.

Optional: call `get_usage` and report **videos used, plan cap, and remaining videos this cycle**. Do not convert that into money.

## After setup

Once `whoami` succeeds on a paid account, they can generate. Use `ai-video-generation` for any-model clips, `ugc-generation` for talking-head UGC ads, and `brand-video` when the brief is on-brand.
