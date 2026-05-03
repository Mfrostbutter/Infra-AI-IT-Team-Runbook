# Model Routing Policy

Specialists are tagged with a default model tier. The parent (main agent) can override at invocation time. **No specialist file declares opus as a default.** Opus is parent-invoked, incident-only.

## Tier definitions

### Tier 1 — Haiku
Routine reads, deterministic checks, classification, well-shaped API calls (curl + JSON), single-secret reads, inventory enumerations. Cheapest, fastest. Default for read-heavy specialists.

### Tier 2 — Sonnet
Planned changes, runbook execution, multi-step coordination across one domain, triage with nuance, hypothesis generation. Default for change-prone or reasoning-heavy specialists.

### Tier 3 — Opus
**Only used when the parent explicitly overrides** an agent invocation to opus. Reserve for:

- Confirmed active compromise with multi-domain blast radius
- Cluster-level outage with conflicting telemetry after initial diagnostics
- More than two failed remediation attempts where root cause is still unclear
- Ambiguous incident where containment vs. recovery tradeoffs require careful reasoning

## Per-agent defaults

| Agent | Default | Notes |
|---|---|---|
| `infra-router` | haiku | pure classification, no execution |
| `infra-vault` | haiku | reads dominate; rotations escalate to sonnet |
| `infra-edge` | haiku | DNS / tunnel / access ops are templated |
| `infra-mesh` | haiku | peer / route reads dominate |
| `infra-flow` | haiku | inventorying jobs and runs is mechanical |
| `infra-docs` | haiku | journal-driven doc updates are templated |
| `infra-vuln-management` | haiku | CVE matching and inventory lookups are mechanical |
| `infra-lan` | sonnet | dual-resolver consistency, mixed API + SSH discipline |
| `infra-substrate` | sonnet | cross-host coordination, storage constraints, ProxyJump |
| `infra-telemetry` | sonnet | correlation, hypothesis generation, observability |
| `infra-security` | sonnet | confidence-leveled triage and hardening reasoning |
| `infra-threat-detection` | sonnet | telemetry correlation and hypothesis generation |

7× haiku, 5× sonnet, 0× opus. Opus is parent-invoked, incident-only.

## Escalation triggers (parent overrides up)

- More than two failed remediation attempts on the same fault
- Conflicting telemetry across domains
- Unclear root cause after initial diagnostics
- Production outage or security-sensitive impact
- Active incident requiring containment-first reasoning under uncertainty

## Downshift triggers (parent overrides down)

- Stable, known runbook with no decision points
- Read-only checks
- Repetitive inventory or verification calls

## Required output per task

Each specialist's task summary should include:

1. **Risk class:** A (read), B (change), C (high-risk / incident)
2. **Tier used:** haiku / sonnet / opus
3. **Why this tier was sufficient** (one line)

## Safety invariants

- Confirmation gates are mandatory for writes, restarts, and destructive actions (see `confirmation-gate.md`)
- Secrets remain masked in agent output (first 4 chars + `…`)
- No destructive operations without explicit user approval
- No specialist auto-escalates its own tier; the parent decides
