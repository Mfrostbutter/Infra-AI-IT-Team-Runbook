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

## Schema validation

The entry shape in [`policies/audit-trail.md`](../policies/audit-trail.md) is a contract, not a suggestion. Validate at the write boundary, before the entry lands:

- **Required fields present.** `ts`, `agent`, `tier`, `action`, `target`, `effect`, `gate_ack`, `outcome`. A write missing any of these is rejected, not coerced.
- **Enums hold.** `tier ∈ {haiku, sonnet, opus}`, `outcome ∈ {success, failure, partial}`, `gate_ack ∈ {y, n, auto}`. `auto` only on a break-glass entry.
- **`ts` is ISO-8601 UTC.** One timezone, always. Cross-host correlation falls apart the first time two agents log in local time.
- **Secret scan on `action` and `effect`.** Reject any entry whose free-text fields match a token/key pattern. The journal is append-only, so a leaked secret in an entry is a leaked secret forever.

Validation lives in the journal endpoint, not in each agent. An agent that has to remember the schema is an agent that will get it wrong.

## Query examples

The journal earns its keep at read time. The shapes you will run weekly:

```bash
# What did the team do in the last 24h, grouped by agent
GET <JOURNAL_URL>/events?since=<iso-24h-ago>

# Everything one specialist touched this week
GET <JOURNAL_URL>/events?agent=infra-edge&since=<iso-7d-ago>

# Only the writes that failed or half-landed (incident hunting)
GET <JOURNAL_URL>/events?outcome=failure,partial&since=<iso>

# Every change to one target during an incident window
GET <JOURNAL_URL>/events?target=<host-or-zone>&since=<start>&until=<end>
```

The daily digest in [`agents/infra-docs.md`](../agents/infra-docs.md) is just the first query, grouped and written to `changelog.md`.

## Integration with telemetry during IR

During an incident, the journal and the SIEM answer different halves of one question. The SIEM shows what *happened* on the hosts. The journal shows what the *team changed*. Correlated on a time window, they separate "the agent did this" from "something else did this."

`infra-telemetry` reads the journal during triage: pull every team change in the incident window, line it up against the SIEM events, and the gap between them is the lead. An alert at 02:14 with no journal entry near it is not the team's doing. An alert at 02:14 with an `infra-edge` write at 02:13 is a change to roll back first and ask questions after.

This is also why the drift sweep in [section 12](12-drift-detection.md) stores its rows in the same audit store: unauthored change (drift) and authored change (journal) are queryable side by side.

## Backup discipline

The journal is the team's long-term memory. Losing it loses every "why did we do that" answer you will ever need.

- **Append-only at the storage layer**, not just by convention. No agent and no operator gets a delete primitive on the journal.
- **Backed up on the same cadence as your most important datastore.** If you snapshot the substrate weekly, the journal rides along.
- **Corrections are entries, not edits.** A wrong entry is fixed by appending a correction that references the original `ts`. The original stays.
- **Survives a host rebuild.** The journal lives somewhere that is not the box most likely to be the subject of its entries.

## Status

Core content complete. Pairs with [section 12 · drift detection](12-drift-detection.md), which checks reality the journal did not author.
