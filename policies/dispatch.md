# Dispatch Policy (Parent-Side Routing)

Specialist agents do not reliably dispatch to other specialists. Routing happens at the **parent** level — the main agent or harness reading the user's intent. This document is the rule-set the parent uses to pick the right specialist.

## Specialist menu

| Specialist | Owns |
|---|---|
| `infra-mesh` | overlay network: peers, groups, routes, policies, setup keys |
| `infra-vault` | secrets: machine identities, env injection, rotations |
| `infra-substrate` | compute: VMs, containers, SSH topology, host lifecycle |
| `infra-edge` | public surface: DNS, tunnels, access policies, zone management |
| `infra-lan` | internal network: switches, APs, internal resolvers |
| `infra-telemetry` | observability: logs, alerts, agent enrollment, IR triage |
| `infra-flow` | workflow runtime: scheduled jobs, webhooks, automation graphs |

## Routing rules

1. **Single domain → one specialist.** Match the request's primary surface to the table above.
2. **Multi-domain incidents** default to `infra-substrate` as coordinator unless the obvious root cause is edge or mesh.
3. **Security-shaped requests** (incident, exposure, hardening, suspicious telemetry) → `infra-telemetry` first. If scope demands deeper IR, the parent invokes additional specialists as peers.
4. **Health checks and diagnostics** of a service belong to the infra specialist that owns its domain, not to a separate "executor" agent.
5. **Reads are cheap, writes have gates.** Read paths can chain freely. Write paths stop at the confirmation gate.

## Decision-only router (optional)

For teams that want a single entry point, `infra-router` is a decision-only routing wrapper. It returns `{specialist, tier, reason}`. The parent invokes the recommended specialist directly. The router never executes domain operations itself.

## Tier escalation

Each specialist has a default model tier (see `model-routing-policy.md`). The parent escalates by overriding the tier at invocation time when triggers are met. **No specialist auto-escalates its own tier.** Escalation is a parent-side decision, always.

## Why parent-side

A specialist that thinks it can dispatch its peers becomes a router. A router that owns a domain stops being a router. The split collapses back into a monolith. Keep dispatch above the specialist line.
