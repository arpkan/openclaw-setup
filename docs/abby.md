# Abby (openclaw-gateway)

The primary AI agent. Handles job search, LinkedIn, document work, and general tasks.

## Identity

| Property | Value |
|---|---|
| Container | `openclaw-openclaw-gateway-1` |
| Image | `openclaw-super-abby:v1` |
| Gateway | `ws://localhost:18789` |
| noVNC | http://localhost:6080/vnc.html |
| Config | `/home/arpkan/.openclaw/openclaw.json` |
| Workspace | `/home/arpkan/.openclaw/workspace/` |
| Telegram bot | `@[Abby's bot]` |
| Default model | `openrouter/minimax/minimax-m2.5` (MiniMax) |

## Available models

| Alias | Model ID |
|---|---|
| MiniMax (default) | `openrouter/minimax/minimax-m2.5` |
| Gemini | `google-vertex/gemini-3-flash-preview` |
| Gemini Pro | `google-vertex/gemini-3.1-pro-preview` |
| Sonnet | `openrouter/anthropic/claude-sonnet-4.6` |

Switch via Telegram: `/model Gemini`

## Skills

| Skill | Purpose |
|---|---|
| `browser` | Chromium browser control |
| `scheduler` | Cron job scheduling |
| `reader` | Read documents and web pages |
| `office-suite` | LibreOffice (spreadsheets, docs, slides) |
| `python` | Run Python scripts |
| `nano-banana-pro` | Google Places integration |
| `goplaces` | Location/maps lookup |
| `notion` | Notion pages and databases |
| `sag` | SAG integration |
| `brave-search` | Web search via Brave |
| `linkedin` | LinkedIn job search and profile management |

### LinkedIn skill

- Location: `/home/arpkan/.openclaw/workspace/skills/linkedin/SKILL.md`
- Browser-based: job search, profile view/edit, posting
- Login: manual via noVNC — open linkedin.com and log in yourself
- Cookie re-import: `docker exec openclaw-openclaw-gateway-1 node /home/node/.openclaw/import-linkedin-cookies.mjs`

## Plugins

- `telegram` — Telegram bot channel

## Auth profiles

- `google:default` — Google API (api_key mode)
- `openrouter:default` — OpenRouter (api_key mode)
- `google-vertex:default` — Google Vertex AI via ADC

## Telegram settings

- DM policy: `pairing` (must pair before DMing)
- Group policy: `allowlist` (only allowed groups)
- Stream mode: `partial` (streams partial replies)

## Config file

`/home/arpkan/.openclaw/openclaw.json`

After editing, restart Abby:
```bash
docker compose restart openclaw-gateway
# or for a full reload:
sudo systemctl restart openclaw
```
