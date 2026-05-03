# 04 · The Edge

> Your public surface. DNS, tunnels, access policies.

## Why this comes fourth

By now the substrate runs services. The edge is how the world (or you, from outside the mesh) reaches them — without exposing anything to a port-scanner.

What this section establishes:

1. **Token scoping.** One token per surface (DNS, tunnels, access). Least privilege from day one.
2. **DNS automation.** Short TTLs, scripted creates, the convention for which records are public vs. internal.
3. **One service end-to-end.** A tunnel from your edge provider → your private mesh → an internal service, gated by an access policy.

The end state: zero ports exposed to the public internet, and services still reachable from anywhere.

## Choices

- **Reference tool.** Cloudflare (free tier covers Tunnels, Access for one user, DNS, and edge cache). Equivalents: ngrok, Tailscale Funnel, Pomerium, your own reverse proxy + WireGuard.

## Status

Stub. Full section drops next: token scoping templates, DNS automation, the tunnel-walkthrough for one service.
