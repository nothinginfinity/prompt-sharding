# Prompt Sharding

> A distributed trust protocol for AI workflows.
> Segment LLM context across multiple providers so no single model,
> provider, or breach sees the complete picture.

**Status:** Specification v0.1 — open for review
**Author:** [@nothinginfinity](https://github.com/nothinginfinity)
**Built on:** [MMCP](https://github.com/nothinginfinity/m-mcp) · [mmcp-inbox-router](https://github.com/nothinginfinity/mmcp-inbox-router)

---

## The Problem

When you send a sensitive document, codebase, or conversation to an AI provider, that provider receives the full context. This means:

- A single API breach exposes everything
- A single provider can train on your complete data
- A single prompt injection can corrupt the entire workflow
- You are placing unconditional trust in one vendor’s security posture

For personal use this is a nuisance. For enterprise use — healthcare, legal, finance, government — it is a blocker.

---

## The Insight

AI workflows are sequential. A research pipeline, a code review, a game turn, a document analysis — all of these are **chains of turns**. Each turn only needs a slice of the total context to do its job.

**Prompt Sharding** routes each turn to a different AI provider. No provider ever receives the assembled whole. The complete picture exists only at the ledger — a durable, tamper-evident record that the *protocol* controls, not any single vendor.

---

## How It Works

```
Full Task (e.g. “Review this contract”)
  │
  ├── Shard 1 → Alice (Perplexity)  — structural analysis
  ├── Shard 2 → Bob   (Claude)       — risk identification
  └── Shard 3 → Carol (ChatGPT)     — plain-language summary
         │
         └── Ledger (GitHub)  — assembles receipts, never full context
                  │
                  └── Router      — routes shards, never combines them
```

Each AI agent:
- Receives **only its shard** — the slice it needs for its role
- Writes its output to **its own outbox** in the ledger
- Never reads another agent’s context or output directly
- Is **replaceable** — swap any provider without changing the protocol

---

## Security Properties

| Property | Description |
|---|---|
| **Context segmentation** | No provider sees the full input |
| **Output isolation** | No provider sees another agent’s output |
| **Provider independence** | Any LLM can be a shard agent |
| **Tamper evidence** | Git commit history is the audit log |
| **Breach containment** | Compromising one provider yields one shard only |
| **Training data isolation** | Providers train on fragments, never the whole |

### What it protects against

- AI provider breach → attacker sees one shard only
- Training data exposure → provider trains on a meaningless fragment
- Prompt injection → contained to one agent’s shard
- Provider surveillance → no vendor has the full picture
- Vendor lock-in → swap any agent without protocol changes

### What it does NOT protect against

- **Ledger compromise** — the GitHub repo holds routing rules and assembled outputs. Hardening the ledger (encryption at rest, signed envelopes, HSM-backed secrets) is the responsibility of the [`stone-vault`](https://github.com/nothinginfinity/stone-vault) layer.
- **Shard correlation attacks** — a sophisticated adversary with access to multiple providers could attempt to correlate outputs. Shard design should minimize cross-shard semantic overlap.
- **Router compromise** — the routing logic must be auditable and tamper-evident. See [`mmcp-inbox-router`](https://github.com/nothinginfinity/mmcp-inbox-router).

---

## Docs

| Document | Description |
|---|---|
| [Shard Schema](docs/shard-schema.md) | `shards.json` format and field reference |
| [Envelope Format](docs/envelope-format.md) | Signed envelope spec for shard delivery |
| [NETWORK.md Template](docs/NETWORK-template.md) | Agent registry format |
| [Enterprise Guide](docs/enterprise.md) | From personal use to production hardening |
| [Threat Model](docs/threat-model.md) | Full attacker model and mitigations |
| [Roadmap](docs/roadmap.md) | Version milestones |

---

## Quick Start

See [docs/quickstart.md](docs/quickstart.md) for a working two-agent example using two Perplexity Spaces and a GitHub repo as the ledger.

---

## Reference Implementation

Runs entirely on an iPhone — no server required:

| Repo | Role |
|---|---|
| [`m-mcp`](https://github.com/nothinginfinity/m-mcp) | Envelope protocol |
| [`mmcp-inbox-router`](https://github.com/nothinginfinity/mmcp-inbox-router) | Router engine |
| [`stone-vault`](https://github.com/nothinginfinity/stone-vault) | Credential isolation |
| [`pocket-agent-engine`](https://github.com/nothinginfinity/pocket-agent-engine) | Orchestrator |

---

## License

MIT — use freely, build on it, give credit.
