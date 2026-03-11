# Architecture

## Overview

```
Windows (boots)
  └── WSL2 (Ubuntu)
        └── systemd
              └── openclaw.service        ← starts on WSL2 boot
                    └── docker compose up
                          ├── openclaw-gateway  (Abby)
                          └── openclaw-beta     (Beta)
```

## Components

### WSL2
Windows Subsystem for Linux 2 — runs a full Ubuntu environment inside Windows. All Docker containers and services run here.

### systemd
Linux service manager running inside WSL2. Starts the Docker Compose stack automatically when WSL2 boots.

Service file: `/etc/systemd/system/openclaw.service`

### Docker Compose
Manages two always-on containers. Defined in `/home/arpkan/openclaw/docker-compose.yml`.

### Abby (`openclaw-gateway`)
The primary AI agent. Has custom skills including LinkedIn and job search.

| Property | Value |
|---|---|
| Image | `openclaw-super-abby:v1` |
| Gateway port | 18789 |
| Bridge port | 18790 |
| noVNC | http://localhost:6080/vnc.html |
| Display | :99 |
| VNC port | 5900 |
| Config | `/home/arpkan/.openclaw/` |
| Default model | `openrouter/minimax/minimax-m2.5` |

### Beta (`openclaw-beta`)
Secondary AI agent. Built-in skills only. Has cron scheduling enabled.

| Property | Value |
|---|---|
| Image | `openclaw-beta:v1` |
| Gateway port | 18889 |
| Bridge port | 18890 |
| noVNC | http://localhost:6081/vnc.html |
| Display | :100 |
| VNC port | 5901 |
| Config | `/home/arpkan/.openclaw-beta/` |
| Default model | `google-vertex/gemini-3-flash-preview` |

## Networking

Both containers use `network_mode: host` — they share the host's network stack directly. This means:
- No port mapping overhead
- Containers can reach each other via `localhost`
- Beta can connect to its own gateway at `ws://localhost:18889`

## Shared volumes

| Host path | Container path | Purpose |
|---|---|---|
| `/home/arpkan/.openclaw/` | `/home/node/.openclaw` | Abby's config and workspace |
| `/home/arpkan/.openclaw-beta/` | `/home/node/.openclaw` | Beta's config and workspace |
| `/mnt/c/Users/k-ar/OneDrive/Desktop` | `/home/node/.openclaw/workspace/Desktop` | Windows Desktop (shared) |
| `/home/arpkan/openclaw/gcp-credentials.json` | `/home/node/gcp-credentials.json` | Google Cloud service account (read-only) |

## Models available

| Alias | Model ID | Provider |
|---|---|---|
| Gemini | `google-vertex/gemini-3-flash-preview` | Google Vertex |
| Gemini Pro | `google-vertex/gemini-3.1-pro-preview` | Google Vertex |
| MiniMax | `openrouter/minimax/minimax-m2.5` | OpenRouter |
| Sonnet | `openrouter/anthropic/claude-sonnet-4.6` | OpenRouter |

Switch models via Telegram: `/model Sonnet`
