# 06 · The Telemetry

> Single-node observability. Default ruleset, agent enrollment, the read-mostly domain.

## Why this comes sixth

The substrate runs services. The edge exposes a few. The LAN connects everything. Now you need to *see* it.

The minimum viable observability layer for a personal stack:

1. **One SIEM / log manager.** Wazuh-class. Single node is fine.
2. **Agents on every host.** VMs, containers, your operator workstation. Default ruleset out of the box.
3. **Read-mostly.** Telemetry is the domain agents query the most and mutate the least.

What you do **not** need at this scale: a multi-tier ELK cluster, a paid SIEM, a SOC. One SIEM, sane defaults, a weekly look at the alert volume.

## Choices

- **Reference tool.** Wazuh (free, open-source). Equivalents: Splunk Free, Loki + Grafana, OpenSearch.

## Status

Stub. Full section drops next: single-node Wazuh deploy, agent enrollment script, default ruleset notes, the weekly review pattern.
