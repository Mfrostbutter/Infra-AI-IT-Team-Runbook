# 03 · The Substrate

> Compute and SSH topology. The shape of the box and how you reach it.

## Why this comes third

Mesh and vault are infrastructure for *getting to* your services. The substrate is the *services themselves* — the VMs and containers that do the work.

This section establishes:

1. The hypervisor (Proxmox-class) and the cluster shape (one node is fine; multi-node is not required)
2. The **API token** for programmatic ops (created once, scoped narrowly)
3. The **SSH key + ProxyJump** topology — `~/.ssh/config` is the operator-side source of truth
4. A **naming convention** you can actually remember (services not numbers)

## Choices

- **Reference tool.** Proxmox VE (free, hypervisor + LXC + KVM). Equivalents: bare Docker on a beefy host, Incus, or a managed cloud equivalent.
- **One node vs. cluster.** Start with one. Cluster when you have a real reason.

## Status

Stub. Full section drops next: API token scoping, SSH key generation, ProxyJump pattern, naming convention with examples.
