# 00 · Prerequisites

> Read this before anything else. Skipping it is the most common reason readers stall on day one.

## What you need before the runbook is useful

| Requirement | Why |
|---|---|
| One Linux box (homelab, NUC, VPS) | Every domain in this runbook runs *something* on it |
| One owned domain | The edge layer needs DNS records you control |
| SSH literacy | You will edit `~/.ssh/config`, ProxyJump, and run shell commands as root |
| A vault provider account | Secrets cannot live on disk. Free tiers are fine |
| A mesh provider account | Self-host or managed — both are fine. Free tier is fine |
| An edge provider account | Cloudflare's free tier covers everything in this runbook |
| A `~/.ssh/config` you maintain | Internal DNS plus aliases is how the agents talk to your hosts |

## What you do not need

- Kubernetes
- A public IP (you will tunnel through your edge provider)
- Multiple servers (you can run the entire stack on one box at first)
- An "AI platform" subscription (you only need API access from your model provider)

## How long this takes

| Phase | Calendar time |
|---|---|
| 0 (this) | 30 minutes |
| 1–6 (substrate buildout) | 1–2 weekends |
| 7 (the agent monolith) | 1 weekend |
| 8 (the split) | 1 weekend, after at least 30 days running the monolith |
| 9–11 (memory, journal, anti-patterns) | continuous |

## Status

This section is a stub. Full content drops next.
