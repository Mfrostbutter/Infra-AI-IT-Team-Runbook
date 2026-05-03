# 09 · The Memory + Proactive Layer

> Vector store, daily digest, weekly memory promotion. How the team gets smarter without you typing more.

## Why a memory layer at all

A specialist agent that does not remember its own past runs is a stateless calculator. It will:

- Re-run the same diagnostic you ran last week
- Forget the gotcha that bit you the last three times
- Miss the pattern across last month of telemetry alerts

A memory layer changes the team from a stateless tool into an operator that **gets better the more you use it**.

## What this section covers

1. A vector store (Qdrant-class) the agents query for prior runs, decisions, and gotchas
2. A daily digest job that summarizes what the team did and surfaces what changed
3. A weekly memory-promotion ritual: review the digest, promote durable facts to long-term memory, archive the rest
4. The proactive layer: scheduled jobs that run reads and *suggest* changes (the agents propose, the operator disposes)

## Choices

- **Vector store.** Qdrant, Weaviate, pgvector. Whichever your stack already runs.
- **Embedding model.** OpenAI `text-embedding-3-large` or any equivalent dense embedding.

## Status

Stub. Full section drops next.
