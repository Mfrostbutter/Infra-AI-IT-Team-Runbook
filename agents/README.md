# Specialist Agents

Each agent owns exactly one infrastructure domain. They are 80–150 lines, runbook-embedded, and load only when the parent invokes them.

| Agent | Default model | Domain |
|---|---|---|
| [`infra-router`](infra-router.md) | haiku | decision-only routing wrapper |
| [`infra-mesh`](infra-mesh.md) | haiku | overlay network |
| [`infra-vault`](infra-vault.md) | haiku | secrets and machine identity |
| [`infra-substrate`](infra-substrate.md) | sonnet | compute and SSH topology |
| [`infra-edge`](infra-edge.md) | haiku | public surface |
| [`infra-lan`](infra-lan.md) | sonnet | internal network |
| [`infra-telemetry`](infra-telemetry.md) | sonnet | observability and IR |
| [`infra-flow`](infra-flow.md) | haiku | workflow runtime |
| [`infra-docs`](infra-docs.md) | haiku | journal-driven docs and changelog |
| [`infra-security`](infra-security.md) | sonnet | security triage, exposure review, hardening |
| [`infra-threat-detection`](infra-threat-detection.md) | sonnet | active alert triage, telemetry correlation |
| [`infra-vuln-management`](infra-vuln-management.md) | haiku | CVE prioritization, patch sequencing |

## How specialists are structured

Every specialist follows the same five-block shape:

```
---
frontmatter (name, description, tools, model)
---

You own <domain>.

## Scope          — what this agent owns end-to-end
## Context        — env vars, hosts, vault paths
## Preflight      — the first command, always run before anything else
## Common ops     — the templated read paths
## Safeguards     — what stops at the confirmation gate
## Break-glass    — what to do when the domain is down
## References     — policy files this specialist depends on
```

## Placeholder convention

Public agent specs use placeholders, not real values:

| Placeholder | Replace with |
|---|---|
| `<MESH_FQDN>` | your mesh management URL |
| `<MESH_HOST>` | hostname of the mesh control plane |
| `<EDGE_API_TOKEN>` | scoped edge (DNS/tunnel) API token |
| `<VAULT_HOST>` | hostname of your secrets vault |
| `<HYPERVISOR_HOST>` | your hypervisor (Proxmox, etc.) hostname |
| `<LAN_CONTROLLER>` | your LAN controller URL |
| `<TELEMETRY_HOST>` | your SIEM / observability host |
| `<JOURNAL_URL>` | your central audit journal endpoint |
| `<DOMAIN>` | your owned domain |

Keep your real values in your secrets vault. Inject at runtime. Never paste them into agent prompts.

## Adapting these specs

1. Copy the file into your agent harness (e.g., `~/.claude/agents/`).
2. Replace placeholders with vault-injected env vars in your runtime.
3. Adjust the `tools:` list to your harness's tool names.
4. Test each specialist against its preflight before letting it touch a write path.
