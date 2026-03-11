# OpenClaw Setup Guide

A personal runbook for running OpenClaw AI agents on Windows using WSL2, systemd, and Docker.

## What's running

```
Windows
  └── WSL2 (Ubuntu)
        └── systemd
              └── openclaw.service
                    └── Docker Compose
                          ├── Abby (openclaw-gateway)   → port 18789, noVNC :6080
                          └── Beta (openclaw-beta)      → port 18889, noVNC :6081
```

Two AI agents, each with their own Telegram bot, browser, and config.

## Docs

- [Architecture](docs/architecture.md) — how the pieces fit together
- [Docker Setup](docs/docker-setup.md) — images, containers, compose file
- [Abby](docs/abby.md) — Abby's config, skills, and tools
- [Beta](docs/beta.md) — Beta's config, cron, memory, and tools
- [Troubleshooting](docs/troubleshooting.md) — issues hit and how they were fixed

## Quick reference

| Action | Command |
|---|---|
| Start all | `sudo systemctl start openclaw` |
| Stop all | `sudo systemctl stop openclaw` |
| Restart all | `sudo systemctl restart openclaw` |
| Restart Beta only | `docker compose restart openclaw-beta` |
| View Abby logs | `docker logs openclaw-openclaw-gateway-1` |
| View Beta logs | `docker logs openclaw-openclaw-beta-1` |
| Rebuild Abby image | `cd ~/openclaw && docker build -t openclaw-super-abby:v1 .` |
| Rebuild Beta image | `cd ~/openclaw && docker build -f Dockerfile.beta -t openclaw-beta:v1 .` |
