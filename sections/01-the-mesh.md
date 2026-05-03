# 01 · The Mesh

> A peer-to-peer overlay network is the precondition for everything else.

## Why this comes first

Every other domain assumes you can reach your hosts privately, by name, from anywhere. That is what a mesh gives you.

Without the mesh:

- You expose admin surfaces (SSH, API, dashboards) to the public internet, or
- You hairpin everything through a VPN that is itself a single point of failure, or
- You can only operate from your home network

With the mesh, your laptop, your hypervisor, your containers, your phone — everything — is on the same private network with stable hostnames. The agent team is built on this assumption.

## Choices

- **Self-hosted vs. cloud-managed.** Both work. Self-hosted is one more thing to keep alive. Cloud-managed has a free tier that scales to small teams.
- **WireGuard-based vs. legacy VPN.** WireGuard. Always.
- **Reference tool.** NetBird. Equivalents: Tailscale, ZeroTier, headscale (self-hosted Tailscale control plane).

## Status

Stub. Full section drops next: setup keys, peer enrollment, group/route patterns, and the first peer.
