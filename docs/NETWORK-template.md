# NETWORK.md Template

Every prompt-sharding ledger repo contains a `NETWORK.md` at the root.
This is the agent registry — the single source of truth for who lives where.

## Format

```markdown
# Agent Network

> Registry for this prompt-sharding ledger.
> Updated by the router when agents are added or removed.

## Agents

| Name | Role | Inbox | Outbox | Provider | Status |
|---|---|---|---|---|---|
| `router.mmcp` | Router | — | — | mmcp-inbox-router | ✅ active |
| `alice.mmcp` | Shard agent | `spaces/alice.mmcp/inbox.md` | `spaces/alice.mmcp/outbox.md` | Perplexity | ✅ active |
| `bob.mmcp` | Shard agent | `spaces/bob.mmcp/inbox.md` | `spaces/bob.mmcp/outbox.md` | Claude | ✅ active |
| `carol.mmcp` | Shard agent | `spaces/carol.mmcp/inbox.md` | `spaces/carol.mmcp/outbox.md` | ChatGPT | ✅ active |

## Shard Config

Active config: [`config/shards.json`](config/shards.json)

## Ledger Repo

`<owner>/<repo>` — this file’s repository

## Last Updated

`<ISO 8601 timestamp>` by `router.mmcp`
```

## Rules

- Every agent must be registered here before the router will deliver shards to it
- The `router.mmcp` entry is always first and always present
- `Status` must be `✅ active` for the router to route to an agent
- Provider field is informational — the protocol is provider-agnostic
