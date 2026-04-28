# Enterprise Guide

Prompt Sharding scales from a personal iPhone setup to a HIPAA-compliant enterprise pipeline without changing the protocol. Only the infrastructure tier changes.

## Personal → Enterprise Comparison

| Dimension | Personal (v1) | Enterprise |
|---|---|---|
| **Ledger** | GitHub free tier | GitHub Enterprise + audit log API |
| **Agents** | Perplexity Spaces | Dedicated provider contracts with DPA |
| **Credentials** | PAT in stone-vault | HSM-backed service accounts |
| **Policy enforcement** | Advisory (`policy` field) | Cryptographically signed + verified |
| **Shard encryption** | Plaintext in ledger | AES-256 at rest, key in HSM |
| **Audit trail** | Git commit history | Immutable audit log + SIEM integration |
| **Shard design** | Manual | Automated shard optimization pipeline |
| **Compliance** | None | HIPAA / SOC 2 / ISO 27001 ready |

## GitHub Enterprise Integration

Prompt Sharding is designed to integrate naturally with GitHub Enterprise:

1. **Ledger = GitHub Enterprise repo** with branch protection, required reviews, and audit log API enabled
2. **Router = GitHub Actions** running on private runners inside the enterprise network perimeter
3. **Agent inboxes = GitHub Environments** with environment-level secrets and approval gates
4. **Receipts = GitHub Issues** with structured metadata for compliance reporting
5. **Audit trail = GitHub Enterprise audit log** — every shard delivery is a signed commit

## Compliance Use Cases

### Healthcare (HIPAA)

A patient record is never sent to a single AI provider in full. Demographic data, clinical notes, and billing codes are sharded to three separate providers under BAAs. No provider holds a complete PHI record. The ledger holds only routing metadata, not the record itself (v2+).

### Legal (Attorney-Client Privilege)

A contract is reviewed by three AI providers, each seeing only its assigned role slice. No provider has enough context to reproduce the privileged communication. The assembled receipt is generated inside the firm’s private infrastructure.

### Finance (SOX / FINRA)

Financial models are reviewed with each provider seeing only the data necessary for its analytical role. The full model is never reconstructable from any single provider’s training data.

### Government / Defense

Classified document analysis routes each classification level to a provider with the appropriate clearance level. No single provider sees above its cleared level.

## GitHub Partnership Opportunity

Prompt Sharding uses GitHub as the neutral ledger — the trusted third party that no AI provider controls. This positions GitHub as:

- The **compliance backbone** for enterprise AI workflows
- The **audit trail provider** for regulated industries
- The **neutral coordinator** in multi-provider AI pipelines
- The **enforcement layer** for AI usage policies

This is a natural extension of GitHub’s existing role in software supply chain security (SBOM, Dependabot, secret scanning) applied to the AI workflow layer.
