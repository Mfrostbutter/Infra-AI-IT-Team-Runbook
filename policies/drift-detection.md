# Drift Detection Policy

Documentation drifts from reality. Not because anyone is careless, but because the world changes through paths the team did not author: a crash, a hand-made container, a direct vault edit. The [journal](audit-trail.md) records what the team did. This policy defines the scheduled sweep that measures the gap between your declared baseline and what is actually running.

## The contract

A drift sweep runs on a schedule, compares a declared baseline against observed reality across several independent detectors, and produces a report plus structured audit rows. By default it **changes nothing**. It detects, it reports, it proposes. Correction is a separate, explicit, tiered decision.

An auditor that can silently rewrite the thing it audits is not an auditor. Keep detection and correction on opposite sides of a gate.

## Detectors

Each detector is independent and declares its own expected state. Generalize these to your stack:

| Detector | Baseline | Observation | Finding when |
|---|---|---|---|
| `reachability` | declared services + datastores/collections | TCP / health probe, datastore listing | expected thing is unreachable or absent |
| `docs` | `last_updated` stamp + max age | the stamp in each canonical doc | stamp older than the max age, or missing |
| `registry` | the agent registry doc | deployed agent spec files | a deployed agent is undocumented (or vice-versa) |
| `prompts` | committed hash baseline + canary token | a fresh hash of every spec | hash moved, or canary token missing |
| `secrets` | a manifest of expected secret **names** | live secret names at a vault path | a name is missing or undocumented (names only, never values) |

## Severity

| Severity | Meaning | Raises alarm |
|---|---|---|
| `high` | a critical service down, a missing datastore/collection, vault unreachable | yes |
| `medium` | a changed prompt, a missing canary, a missing expected secret name | yes |
| `low` | a stale or unstamped doc, a non-critical service down, an undocumented agent | yes |
| `info` | a live-but-undocumented item (extra collection, extra secret name) | no |

A run is `clean` only when no `low`/`medium`/`high` finding exists. `info`-only is reported but clean: discovery, not drift.

## Correction tiers

1. **Report only (default).** Detect, write the report and audit rows, attach a propose-only remediation note to every finding. Touch nothing.
2. **Apply-safe (opt-in).** Additive self-heal only:
   - Append a discovered (live-but-undocumented) item to a `discovered:` block in the baseline for human review.
   - Re-baseline the prompt-integrity hashes, after the change is confirmed intentional.

There is no third tier. The sweep never edits a canonical doc, never touches infrastructure, never reads or writes a secret value, never auto-resolves a `medium`/`high` finding.

## Hard rules

1. **Never invent state.** Every fact in a finding comes from a live observation this run or the declared baseline. If a thing cannot be observed (no vault identity on this host, partial checkout), the detector returns `skip`, not a false flag.
2. **Reality is the source of truth for docs; the team is the source of truth for infra.** When a doc disagrees with reality, fix the doc (via `infra-docs`, gated). When reality is wrong, report it; the owning infra specialist fixes it. The auditor never crosses into a domain it does not own.
3. **Names, never values.** The secrets detector compares names against a manifest. A secret value never enters a finding, a report, the audit store, or a chat summary.
4. **Propose, do not autonomously resolve.** Real findings get a remediation note and wait for the operator or a follow-up reconciliation. Only additive, reviewable self-heal is automatic.
5. **Degrade, do not lie.** A detector that cannot fully run on the current host reports `skip` with a note, not `clean`.

## Where findings go

- A dated markdown report under your knowledge base / audits path.
- One structured row per detector in the audit store, queryable over time next to the journal.
- An optional one-message summary to the operator (`<CHAT_WEBHOOK>`): clean check, or a finding count with the top few inlined.

## Relationship to the journal

The journal and the drift sweep write to the same audit store and answer two halves of one question:

- **Journal:** what did the team change, and did it land? (authored change)
- **Drift sweep:** what changed that the team did not author, and what no longer matches the docs? (unauthored change)

Neither replaces the other. Run both.

## References

- [`audit-trail.md`](audit-trail.md) — the journal and audit store
- [`confirmation-gate.md`](confirmation-gate.md) — the gate that doc corrections pass through
- [`../sections/12-drift-detection.md`](../sections/12-drift-detection.md) — the narrative walkthrough
- [`../agents/infra-docs.md`](../agents/infra-docs.md) — the specialist that reconciles findings
