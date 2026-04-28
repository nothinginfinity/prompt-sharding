# Agent Network

> Registry for this prompt-sharding reference ledger.
> Updated by the router when agents are added or removed.

## Agents

| Name | Role | Inbox | Outbox | Provider | Status |
|---|---|---|---|---|---|
| `router.mmcp` | Router | — | — | mmcp-inbox-router | ✅ active |
| `alice.mmcp` | Shard agent 1 | `spaces/alice.mmcp/inbox.md` | `spaces/alice.mmcp/outbox.md` | Perplexity | ✅ active |
| `bob.mmcp` | Shard agent 2 | `spaces/bob.mmcp/inbox.md` | `spaces/bob.mmcp/outbox.md` | Claude | ✅ active |
| `carol.mmcp` | Shard agent 3 | `spaces/carol.mmcp/inbox.md` | `spaces/carol.mmcp/outbox.md` | ChatGPT | ✅ active |

## Shard Config

Active config: [`config/shards.json`](config/shards.json)

## Ledger Repo

`nothinginfinity/prompt-sharding`

## Last Updated

`2026-04-28T15:00:00Z` by `router.mmcp`
