# Confirmation Gate

Before executing any write, restart, or destructive action, the agent stops and requests confirmation in this exact format:

```
CONFIRMATION REQUIRED
Command: <exact command>
Targets: <host/service/account/zone>
Effect: <plain English description of what changes>
Proceed? (y/n)
```

Reads and diagnostics never require confirmation. Only writes, restarts, and destructive operations do.

## Universally gated actions

1. `systemctl restart` / `stop` of any production unit
2. Container or VM `stop` / `destroy` / `migrate` (Proxmox `pct`/`qm`, Docker, etc.)
3. Any secret write (vault `set`, env update)
4. DNS record create / update / delete
5. Tunnel or Access policy create / update / delete
6. Mesh route, group, or policy delete or destructive edit
7. Mesh setup-key generation (key material is sensitive)
8. LAN config change (port override, VLAN change, firmware flash, reboot)
9. File edits to systemd units, package configs, or hypervisor configs
10. Edits to `~/.ssh/config` on any operator workstation
11. DNS reload on a shared resolver (affects all clients immediately)
12. `git push` of any commit touching infra config
13. SIEM or LAN rule mutation initiated by a security agent

## Why a gate at all

Agents are fast. Fast agents make mistakes at the speed they think. A confirmation gate is the seam where a human reads the *targets* and the *effect* one more time before the change lands. The cost is a half-second of friction. The benefit is one less midnight rollback.

## Anti-patterns

- Auto-approving the gate from the prompt ("just say yes to everything")
- Burying the change inside a multi-step plan and approving the plan, not the change
- Letting an agent gate itself by running its own approval check
