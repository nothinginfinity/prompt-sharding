# Threat Model

## Attacker Profiles

### Attacker A: Compromised AI Provider

**Scenario:** One AI provider (e.g. the service hosting `bob.mmcp`) is breached. The attacker gains full access to everything sent to and received from that provider.

**Without prompt sharding:** Attacker sees the full task, full context, full conversation history.

**With prompt sharding:** Attacker sees only Bob’s shard — one role-isolated slice of the task. The shard’s `role-prompt` ensures it contains the minimum necessary context. The full task is never reconstructable from a single shard.

**Residual risk:** If the attacker can also breach the ledger, they can correlate shards. Ledger hardening (see `stone-vault`) is the mitigation.

---

### Attacker B: AI Provider Training Pipeline

**Scenario:** A provider uses user inputs for model training (per their terms of service).

**Without prompt sharding:** Provider trains on the complete sensitive document.

**With prompt sharding:** Provider trains only on the role-isolated fragment. A shard containing only “identify structural issues in this document” teaches the model nothing about the document’s sensitive content.

**Residual risk:** Metadata (timing, size, sender identity) may still leak information. The `no-persist` policy signals intent but cannot be cryptographically enforced in v1.

---

### Attacker C: Prompt Injection

**Scenario:** A malicious payload is embedded in the task content, attempting to override agent instructions or exfiltrate data.

**Without prompt sharding:** A successful injection in one model can cascade through the entire workflow.

**With prompt sharding:** Injection is contained to one agent’s shard. The router validates output envelopes before assembling receipts. A compromised shard output is flagged, not propagated.

**Residual risk:** The router itself must be injection-resistant. The `mmcp-inbox-router` classifier is deterministic and does not pass raw agent output into routing decisions.

---

### Attacker D: Ledger Compromise

**Scenario:** The GitHub ledger repo is breached — attacker gains read access to all inboxes, outboxes, and routing config.

**Without prompt sharding:** Same as any single-provider breach.

**With prompt sharding:** Attacker can correlate shards and assemble a near-complete picture. This is the primary residual risk of v1.

**Mitigation (v2):** Shard content encrypted at rest using `stone-vault`. Only the router holds decryption keys. Ledger stores ciphertext only.

---

### Attacker E: Shard Correlation

**Scenario:** An attacker with access to multiple providers (or the ledger) attempts to reconstruct the full task by correlating shard outputs.

**Mitigation:**
- Shard roles must be semantically isolated — no shard should be a superset of another
- Task IDs in envelopes are opaque — they do not reveal task content
- `no-cross-shard-read` policy prevents agents from voluntarily sharing context
- In v2: zero-knowledge shard delivery where the router never holds plaintext

---

## Security Tiers

| Tier | Description | Protection Level |
|---|---|---|
| **v1 (current)** | JSON envelopes, advisory policy, GitHub ledger | Protects against provider breach and training exposure |
| **v2** | Signed envelopes, stone-vault secrets, encrypted ledger | Protects against ledger compromise |
| **v3** | ZK shard delivery, HSM-backed router | Protects against shard correlation and router compromise |
