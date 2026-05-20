# PATCHES.md

> Fork patches over upstream `libraz/grok-mcp` to align with our
> `TQZHR/grok2api` proxy at `https://grok2api.praxiway1.workers.dev/v1`.

## What broke

`libraz/grok-mcp` hardcodes xAI's official image/video model names as
defaults:

```typescript
const DEFAULT_IMAGE_MODEL = 'grok-imagine-image-quality';
const DEFAULT_VIDEO_MODEL = 'grok-imagine-video';
```

Our reverse proxy `TQZHR/grok2api` does NOT accept those — it only
recognizes `grok-imagine-1.0` (image) and `grok-imagine-1.0-video`
(video). So when Claude calls the MCP without specifying a model, the
request fails at the proxy with "Model not supported".

There is no env var to override these defaults in upstream.

## Patches

### `src/grok.ts` — defaults + env override

```typescript
const DEFAULT_IMAGE_MODEL = process.env.XAI_DEFAULT_IMAGE_MODEL || 'grok-imagine-1.0';
const DEFAULT_VIDEO_MODEL = process.env.XAI_DEFAULT_VIDEO_MODEL || 'grok-imagine-1.0-video';
```

Both defaults now match `TQZHR/grok2api`'s known good model names.
Override via env vars if the proxy adds new image/video models later.

## Usage

In `~/.claude.json`:

```json
{
  "mcpServers": {
    "grok": {
      "command": "npx",
      "args": ["-y", "github:x-z77/grok-mcp"],
      "env": {
        "XAI_API_KEY": "<your-key>",
        "XAI_BASE_URL": "https://grok2api.praxiway1.workers.dev/v1",
        "XAI_DEFAULT_MODEL": "grok-4.3-beta",
        "XAI_DEFAULT_IMAGE_MODEL": "grok-imagine-1.0",
        "XAI_DEFAULT_VIDEO_MODEL": "grok-imagine-1.0-video"
      }
    }
  }
}
```

`XAI_DEFAULT_IMAGE_MODEL` and `XAI_DEFAULT_VIDEO_MODEL` are optional —
the fork defaults already point to the right names.

Restart Claude Code after changing this file (MCP env vars are loaded at
MCP server startup, not hot-reloaded).

## Syncing upstream

Forks do NOT auto-sync. To pull upstream:

1. Visit `https://github.com/x-z77/grok-mcp` → click "Sync fork"
2. Resolve conflicts (likely in `src/grok.ts`)
3. Verify our 2-line patch survives

If upstream adds env var support for these defaults (worth filing
upstream as a PR), drop our patch entirely.

## Related

- `x-z77/grok2api` — the reverse proxy itself. See its `PATCHES.md` for
  image generation WS protocol fixes that this MCP relies on.
