# Audit Trail Policy

Every specialist writes to a single, central, append-only journal. The journal is the source of truth for "what changed, when, by whom, with what effect." Documentation, changelog, and incident review all read from it.

## The contract

Every **gated action** (see [`confirmation-gate.md`](confirmation-gate.md)) emits exactly one journal entry **after** the action completes. The entry is part of the action — the gate is not satisfied until the entry is written. A failed journal write is retried; a persistently failing journal write triggers an operator alert.

Read paths do not emit entries. Only writes, restarts, and destructive operations.

## Entry shape

```json
{
  "ts":       "2026-05-03T19:42:11Z",
  "agent":    "infra-mesh",
  "tier":     "haiku",
  "action":   "POST /api/setup-keys",
  "target":   "<MESH_FQDN>",
  "effect":   "Generated 24h setup key for new peer 'workstation-laptop'",
  "gate_ack": "y",
  "outcome":  "success",
  "links":    {
    "section":  "01-the-mesh",
    "runbook":  "agents/infra-mesh.md#peer-enrollment"
  }
}
```

| Field | Required | Notes |
|---|---|---|
| `ts` | yes | ISO-8601 UTC |
| `agent` | yes | specialist name (e.g., `infra-edge`) |
| `tier` | yes | `haiku` / `sonnet` / `opus` |
| `action` | yes | one-line, command-shaped |
| `target` | yes | host / service / zone / record |
| `effect` | yes | plain-English description (mirrors the gate's `Effect:`) |
| `gate_ack` | yes | `y` / `n` / `auto` (`auto` only for emergency break-glass) |
| `outcome` | yes | `success` / `failure` / `partial` |
| `links` | optional | references to runbook section, related entries |

Secrets stay masked. Never write a token, key, or credential into `effect` or `action`.

## The journal service

A central append-only log. Implementation is yours to choose; the runbook expects:

- **One canonical endpoint** every specialist knows: `<JOURNAL_URL>/events` (POST, JSON body)
- **Append-only storage.** No edits, no deletes. If an entry was wrong, append a correction entry that references the original.
- **Reachable from every specialist host.** A journal that some agents cannot write to is a journal that lies.

Defensible default implementations:

| Backend | Best for | Notes |
|---|---|---|
| SQLite + tiny HTTP wrapper | one host | simplest; one file, one process |
| Postgres | several hosts | structured queries, reliable |
| n8n / workflow runtime webhook | already running n8n | trades direct SQL for webhook overhead |
| Append-only JSONL on a shared volume | very small setups | fine until concurrency matters |

The runbook assumes the endpoint exists. Your stack picks the storage.

## Read access

The journal is read by:

1. **`infra-docs`** — turns entries into canonical doc updates (changelog, learnings, runbook prose)
2. **`infra-telemetry`** — correlates entries with SIEM events during IR
3. **You, the operator** — `journal tail` for a quick "what did the team do today"

Read access is unrestricted. Write access is **only** through specialist agents emitting after a gate.

## Why central

A per-agent log is a log no one reads. A central journal is the team's collective memory. Agents that come and go, sessions that end and resume, incidents that span days — all of them want one place to look back.

## What does not belong in the journal

- Read-only diagnostics (too noisy)
- Internal reasoning steps (those belong in the agent's task summary, not the journal)
- Secrets in any form
- Free-form chat between operator and agent

## References

- [`confirmation-gate.md`](confirmation-gate.md)
- [`dispatch.md`](dispatch.md)
- [`agents/infra-docs.md`](../agents/infra-docs.md)
