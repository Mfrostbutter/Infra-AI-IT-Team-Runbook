# 11 · Anti-patterns

> Things that look reasonable, then break things. Read this before you ship.

The shortlist (full content drops later):

1. **Hardcoded tokens in agent specs.** A token in a spec file is a token in your git history is a token on someone's laptop. Use placeholder env vars. Inject from the vault at runtime.

2. **Skipping the mesh preflight.** Every specialist's first command is a health check. Skipping it lets the agent operate on stale or wrong assumptions and the failure modes look like the agent is "broken."

3. **Mixing API and SSH writes on the LAN controller.** Pick one path per change. Mixing them is the most common LAN-side incident pattern in homelabs.

4. **Single resolver.** The dual-resolver model is the entire point of the LAN section. One resolver is a 2am pager you didn't know you signed up for.

5. **Splitting too early.** Specialists you build before running the monolith for 30 days will be wrong in ways you cannot predict. The monolith is a measurement instrument, not a temporary mistake.

6. **Confirmation gates approved as a batch.** "Approve plan" is not the same as "approve this change." Gates exist at the change-line, not the plan-line.

7. **Specialist→specialist dispatch.** Specialists do not reliably dispatch to other specialists. Routing happens at the parent. A specialist that thinks it can call its peers becomes a router. A router that owns a domain becomes a monolith. The split collapses.

8. **Auto-escalation to opus.** Opus is parent-invoked, incident-only. A specialist that auto-escalates its own tier is a specialist that bills you for thinking you didn't ask for.

9. **No daily digest.** Without one, the agents make decisions in your name and you find out about them three weeks later when something silently drifts.

10. **No "stop the work" command.** Every agent runtime should have a kill switch the operator knows by heart. Build it before you need it.

## Status

Stub. Full content drops next.
