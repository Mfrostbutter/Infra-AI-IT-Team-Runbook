# Examples

Ready-to-adapt configs and templates referenced by the runbook sections.

Status: stubs. Examples land alongside the section that uses them.

## Planned

| Section | Example |
|---|---|
| 01 — mesh | `mesh/setup-keys.sh` — short-TTL setup-key generator |
| 02 — vault | `vault/folder-layout.md` — `/<entity>/<service>/<KEY>` convention |
| 03 — substrate | `substrate/ssh-config.template` — ProxyJump pattern with naming convention |
| 04 — edge | `edge/dns-create.sh` — scoped-token DNS upsert |
| 05 — lan | `lan/dual-resolver-sync.sh` — mirror Pi-hole custom lists across two nodes |
| 06 — telemetry | `telemetry/agent-enroll.sh` — bulk SIEM agent enrollment |
| 07 — the agent | `agents/monolith.template.md` — the one-agent starting point |
| 08 — the split | already in [`/agents`](../agents) |
| 09 — memory | `memory/daily-digest.sh` — read recent runs, summarize, surface changes |
| Appendix | `cost-breakdown.md` — monolith vs. team monthly API estimate |

## Conventions

- Every shell script is **idempotent**. Re-running with the same inputs produces the same result.
- Every script reads secrets from your vault, never `.env` on disk.
- Every config uses placeholders (`<MESH_FQDN>`, `<DOMAIN>`, etc.) — see [`/agents/README.md`](../agents/README.md).
