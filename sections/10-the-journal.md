# 10 · The Journal

> Every agent logs its work somewhere central. Without the journal, the team has no shared memory.

## Why a journal exists

A specialist that runs and forgets is a stateless tool. A team of specialists that each forget is a chaos of stateless tools. The journal is the seam where every gated action lands one structured entry — append-only, central, queryable.

The journal answers four questions you will care about every week:

1. **What changed?** (effect)
2. **Who changed it?** (agent + tier)
3. **When?** (ts)
4. **Did it actually land?** (outcome)

Without those answers, the daily digest is fiction, the weekly review is hand-waving, and incident triage starts from "I don't know what we did yesterday."

## The contract

Every gated action emits exactly one journal entry **after** the action completes. The gate is not satisfied until the entry is accepted. See [`policies/audit-trail.md`](../policies/audit-trail.md) for the entry shape and the field-by-field schema.

Read paths do not emit entries. Reads are cheap; logging every `pct list` is noise.

## What writes to the journal

Every specialist:

- `infra-mesh`, `infra-vault`, `infra-substrate`, `infra-edge`, `infra-lan`, `infra-telemetry`, `infra-flow`
- Including `infra-docs` itself — when the docs agent updates a runbook section, that update is journaled like any other write.

The router does not write (it doesn't execute).

## What reads from the journal

- **`infra-docs`** — turns the journal into the changelog and `learnings.md`
- **`infra-telemetry`** — correlates with SIEM events during IR
- **You** — `journal tail` for "what did the team do today"

## Implementation choices

The runbook does not prescribe a backend. It prescribes a **contract**: one endpoint, append-only, structured entries. Pick:

| Backend | Best for |
|---|---|
| SQLite + tiny HTTP wrapper | one host, simplest |
| Postgres | multi-host with structured query needs |
| n8n / workflow runtime webhook | already running n8n |
| Append-only JSONL on a shared volume | very small setups |

Whichever you pick, the agent specs do not change — they POST to `<JOURNAL_URL>/events` and stop caring how it's stored.

## Daily digest

`infra-docs` reads the last 24 hours of entries, groups by agent, and writes a date-stamped section to `changelog.md`. The pattern is in [`agents/infra-docs.md`](../agents/infra-docs.md).

## What does **not** belong in the journal

- Read-only diagnostics (noise)
- Internal reasoning (belongs in the agent's task summary, not the audit log)
- Secrets in any form
- Free-form chat between you and the agents

## Status

Stub. Full content drops next: schema-validation patterns, query examples, integration with telemetry during IR, journal-backup discipline.
