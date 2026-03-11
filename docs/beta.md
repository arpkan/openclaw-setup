# Beta (openclaw-beta)

Secondary AI agent. Built-in skills only. Focused on content, scheduling, and automation.

## Identity

| Property | Value |
|---|---|
| Container | `openclaw-openclaw-beta-1` |
| Image | `openclaw-beta:v1` |
| Gateway | `ws://localhost:18889` |
| noVNC | http://localhost:6081/vnc.html |
| Config | `/home/arpkan/.openclaw-beta/openclaw.json` |
| Workspace | `/home/arpkan/.openclaw-beta/workspace/` |
| Telegram bot | `@betaPolymathBot` |
| Default model | `google-vertex/gemini-3-flash-preview` (Gemini) |

## Available models

| Alias | Model ID |
|---|---|
| Gemini (default) | `google-vertex/gemini-3-flash-preview` |
| Gemini Pro | `google-vertex/gemini-3.1-pro-preview` |
| MiniMax | `openrouter/minimax/minimax-m2.5` |
| Sonnet | `openrouter/anthropic/claude-sonnet-4.6` |

Switch via Telegram: `/model Sonnet`

## Skills

| Skill | Purpose |
|---|---|
| `browser` | Chromium browser control |
| `scheduler` | Cron job scheduling |
| `goplaces` | Location/maps lookup |
| `nano-banana-pro` | Google Places integration |
| `notion` | Notion pages and databases |
| `nspost` | NS posting |
| `nscontent` | NS content generation |

## Tools

| Tool | Notes |
|---|---|
| `cron` | Schedule tasks — added to `tools.alsoAllow` |
| `llm-task` | JSON-only LLM tasks for workflow engines |

## Plugins

- `telegram` — Telegram bot channel
- `llm-task` — LLM task runner

## Memory system

Beta wakes up fresh each session but reads these files at the start:

| File | Purpose |
|---|---|
| `workspace/SOUL.md` | Beta's identity and personality |
| `workspace/USER.md` | Who you are and your preferences |
| `workspace/MEMORY.md` | Long-term curated memory (main session only) |
| `workspace/memory/YYYY-MM-DD.md` | Daily session logs |
| `memory/main.sqlite` | Vector memory store |

Beta writes to these files to persist context across sessions. Edit them directly at `/home/arpkan/.openclaw-beta/workspace/`.

## Cron scheduling

Beta can schedule its own tasks using the built-in `cron` tool. Example from the host to list Beta's cron jobs:

```bash
docker exec \
  -e OPENCLAW_GATEWAY_URL=ws://localhost:18889 \
  -e OPENCLAW_GATEWAY_TOKEN=<token> \
  openclaw-openclaw-beta-1 \
  node /app/dist/index.js cron list
```

Or just ask Beta via Telegram: *"Schedule a daily summary at 9am"*

## Telegram settings

- DM policy: `pairing`
- Group policy: `allowlist` — groups must be added to `groupAllowFrom` in config
- Stream mode: `partial`

## Config file

`/home/arpkan/.openclaw-beta/openclaw.json`

After editing, restart Beta:
```bash
docker compose restart openclaw-beta
```

## Sending a message to Beta from the host

```bash
docker exec \
  -e OPENCLAW_GATEWAY_URL=ws://localhost:18889 \
  -e OPENCLAW_GATEWAY_TOKEN=<token> \
  openclaw-openclaw-beta-1 \
  node /app/dist/index.js agent --agent main --message "Your message here"
```
