# 11 · The Drift Sweep + Docs Agent

> The journal records what the team did. The drift sweep finds what changed without the team, and the docs agent reconciles the canonical sources to match.

## The gap the journal cannot see

[Section 10](10-the-journal.md) gives you a perfect record of every gated action the agents took. It is complete for changes that went through an agent. It is blind to everything else:

- A service that died on its own and never came back.
- A container, datastore, or vector collection someone stood up by hand.
- A canonical doc that quietly fell out of date because the change that invalidated it bypassed the team.
- An agent spec that was edited outside the normal flow (drift, or tampering).
- A secret that was renamed or deleted directly in the vault.

None of these emit a journal entry, because none of them went through a gated action. The journal is the team's memory of its own work. Drift detection is the team's check against reality it did not author.

The two are complements. You want both. A journal without drift detection trusts that nothing changes except through the team. That assumption is wrong the first time a service crashes at 3am.

## What a drift sweep checks

The reference auditor runs a handful of independent detectors, each comparing a declared baseline against observed reality. The categories generalize to any stack:

| Detector | Compares | Catches |
|---|---|---|
| Reachability | declared services + datastores vs live TCP / health probes | a service that died, a moved port, a missing vector collection |
| Doc staleness | `last_updated` stamps in canonical docs vs a max age | docs no one has reconciled in N days |
| Agent registry | deployed agent spec files vs the registry doc | a specialist running but undocumented, or documented but gone |
| Secret inventory | secret **names** at a vault path vs a declared manifest (names only, never values) | a secret renamed or deleted out from under its consumers |

Each detector emits structured findings with a severity. A finding is `info` (live but undocumented, usually benign), `low`/`medium`/`high` (a real gap), and the run is `clean` only when no real finding exists. Info-only is reported but does not raise the alarm.

One more check rides the same weekly sweep: a prompt-integrity hash of every agent spec and policy. Mechanically it is just another detector on this schedule, but it belongs to the guardrails story rather than the docs-vs-reality story, so it is covered in [section 12](12-the-guardrails.md). A spec whose hash moved without a matching baseline update is flagged before it runs again.

## Correction is tiered

The default posture is **detect and report only**. Every finding carries a propose-only remediation note. The auditor changes nothing on its own.

A second, opt-in tier (`--apply-safe` in the reference implementation) does **additive self-heal only**:

- Append a live-but-undocumented item (a discovered collection, a reachable service) to a `discovered:` block in the baseline for human review.
- Re-baseline the prompt-integrity hashes after you have confirmed the change was intentional.

That is the whole list. The safe tier never edits a canonical doc, never touches infrastructure, never reads or writes a secret value. The reasoning is in [`policies/drift-detection.md`](../policies/drift-detection.md): an auditor that can silently rewrite the thing it audits is not an auditor.

## Reality is the source of truth, but the auditor does not fix infra

When a doc disagrees with reality, reality wins and the **doc** gets corrected (by [`infra-docs`](../agents/infra-docs.md), through the normal confirmation gate). When reality itself is wrong (a service that should be up is down), the auditor reports it and the relevant infra specialist fixes it. The drift sweep documents and surfaces; it never reaches into a domain it does not own. A docs auditor that restarts services has become a monolith with extra steps.

## Where it runs and what it produces

- **On a schedule.** Weekly is the reference cadence (systemd timer or cron). Often enough to catch a silent failure inside a week, rare enough to stay cheap.
- **From any meshed host.** Reachability, doc, registry, and prompt detectors work from anywhere on the overlay. The secret-inventory detector needs the vault machine identity, so it runs fully only on the host that holds it and degrades to `skip` elsewhere rather than false-flagging.
- **Three outputs per run:**
  1. A dated markdown report (`<DRIFT_REPORT_DIR>/YYYY-MM-DD-drift.md`).
  2. One structured row per detector in your audit store, so drift is queryable over time alongside the journal.
  3. An optional one-message summary to `<CHAT_WEBHOOK>`: clean check or a count of real findings with the top few inlined.

When a sweep finds real drift, the operator (or a follow-up `infra-docs` invocation) reconciles each finding. When it is clean, the green check is itself a signal: the declared baseline still matches the world.

## How it wires to the rest of the team

```
scheduled trigger
   └─ drift sweep (reachability · docs · registry · secrets · prompt-integrity §12)
        ├─ markdown report  ──►  knowledge base / audits
        ├─ audit-store rows ──►  queryable next to the journal
        └─ chat summary     ──►  operator
                                   └─ invokes infra-docs to reconcile real findings
```

The sweep does not dispatch agents. It reports. The parent decides whether a finding becomes an `infra-docs` reconciliation, an infra-specialist fix, or an accepted change folded into the baseline. Routing stays above the specialist line, same as everywhere else in this runbook.

## References

- [`policies/drift-detection.md`](../policies/drift-detection.md) — the contract, severities, and correction tiers
- [`policies/audit-trail.md`](../policies/audit-trail.md) — the journal the sweep stores rows beside
- [`agents/infra-docs.md`](../agents/infra-docs.md) — the specialist that reconciles findings
- [`sections/10-the-journal.md`](10-the-journal.md) — the record the sweep complements
- [`sections/12-the-guardrails.md`](12-the-guardrails.md) — where the prompt-integrity check that rides this sweep is covered
