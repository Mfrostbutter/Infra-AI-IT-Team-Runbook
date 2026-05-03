# 02 · The Vault

> Machine identity and env injection. No `.env` on disk.

## Why this comes second

The mesh gets your hosts talking to each other. The vault gets your services authenticated to each other without anyone hardcoding a token.

The discipline this section enforces:

1. Every service runs under a **machine identity** (universal-auth, not a personal API key)
2. Secrets are **injected at runtime** (`vault-cli run -- <command>`), never written to a `.env` on disk
3. Every secret has a **path** (`/<entity>/<service>/<KEY_NAME>`), never a flat global namespace
4. The agent team follows the same discipline — agents never see secrets, they ask the vault to inject them

## Choices

- **Reference tool.** Infisical. Equivalents: Doppler, HashiCorp Vault, AWS/GCP secret managers.
- **Self-hosted vs. cloud.** Cloud free tier is fine for personal scale.

## Status

Stub. Full section drops next: machine-identity setup, folder layout, `vault-cli run` patterns, secret rotation, break-glass.
