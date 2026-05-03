# 05 · The LAN

> Internal resolvers, internal DNS, why your wifi controller is not your DNS.

## Why this comes fifth

Mesh + edge solves "how do I reach this from outside." The LAN solves "how do I reach this from *inside* without typing IP addresses for the rest of my life."

This section establishes:

1. **Dual resolvers.** Two Pi-hole-class boxes, mirrored. One can die without taking the network down.
2. **Internal DNS for SSH aliases.** `hypervisor.lan`, `vault.lan`, `mesh.lan` — names you can type at 2am.
3. **The API-vs-SSH discipline.** LAN controllers (UniFi, etc.) have an API *and* an SSH surface. Pick one path per change. Mixing them is the most common LAN-side incident pattern.

## Choices

- **Resolver.** Pi-hole, AdGuard Home, Unbound. Two of whichever you pick.
- **LAN controller.** UniFi, OPNsense, MikroTik. Whichever you have. The runbook teaches the *patterns*.

## Status

Stub. Full section drops next: resolver pair setup, custom-list discipline, SSH-alias DNS conventions, controller API patterns.
