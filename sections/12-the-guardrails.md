# 12 · The Guardrails

> The drift sweep tells you a spec changed. The guardrails decide what a compromised, tampered, or misled agent can actually do.

## Three things you are defending against

1. **An agent spec edited outside the normal flow.** Drift, or deliberate tampering.
2. **A live model that has quietly stopped behaving.** The file on disk looks right, the behavior does not.
3. **Hostile instructions riding in on retrieved content.** A web page, a ticket, or a scraped doc that says "ignore your rules."

The drift sweep in [section 11](11-the-drift-sweep.md) catches the first on a schedule. The other two need their own guards.

## Prompt integrity: hash the specs

Hash every agent spec and every policy file, store the hashes in a committed baseline, and re-check them on the weekly sweep. Any spec whose hash moved without a matching baseline update flags immediately. This is the cheapest tamper-evidence you can buy. It needs no model call, runs in milliseconds, and turns "did someone edit an agent?" into a diff instead of a guess.

Re-baselining is a deliberate, reviewed step. You change a spec on purpose, you confirm the change was yours, you re-stamp the hash. An auditor that re-baselines silently is back to trusting the thing it audits.

## The canary: prove the live model still behaves

A hash proves the file on disk is intact. It says nothing about the model that loads it. For that you need a runtime probe.

Seed a known challenge token in the system prompt with an exact expected response. On a schedule, ask the live model the challenge and check the answer verbatim. In the same probe, send a couple of injection attempts the agent is supposed to refuse. Three outcomes matter:

- The canary answers correctly and refuses the injections. The prompt is intact and the defense holds.
- The canary stops firing. The system prompt is not reaching the model the way you think it is.
- An injection succeeds. The defense has a hole, and you find it on your schedule instead of during an incident.

The hash catches a changed file. The canary catches behavior the file cannot show.

## Injection defense: retrieved content is data, never instructions

This is the one rule the whole fleet shares, and the one [OWASP ranks LLM01](https://owasp.org/www-project-top-10-for-large-language-model-applications/). Everything an agent retrieves (tool output, web pages, files, tickets, scraped data) is untrusted data. It is never an instruction, no matter how it is phrased. The hierarchy is fixed: the system prompt and the operator outrank anything that arrives in a payload.

The full rule, with the handling pattern for each retrieval source, lives in [`policies/injection-defense.md`](../policies/injection-defense.md). It is a non-negotiable line in every agent's context, not a suggestion.

## Least privilege caps the blast radius

The guards above are detection and refusal. Least-privilege write paths are containment. If an agent is compromised anyway, it should be able to do very little. Read paths are wide open. Write paths are scoped to one domain, gated behind confirmation, and hold no standing secrets. A specialist that can read everything and change almost nothing is a small blast radius by construction.

## References

- [`policies/injection-defense.md`](../policies/injection-defense.md): the retrieved-content-is-data rule, source by source.
- [`policies/drift-detection.md`](../policies/drift-detection.md): the sweep the prompt-integrity hash rides.
- [`sections/11-the-drift-sweep.md`](11-the-drift-sweep.md): the scheduled audit that runs the hash check.
- [`sections/08-the-split.md`](08-the-split.md): where the least-privilege write paths are defined.
