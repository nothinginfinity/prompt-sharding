# Security Model — Prompt Sharding

> Version: 0.1 | Last updated: 2026-04-28

This document defines what Prompt Sharding protects, what it does not protect,
the honest threat model for v1, and the architectural path toward stronger guarantees.

---

## The Master Key Problem

Prompt Sharding requires a GitHub Personal Access Token (PAT) to read and write
the ledger. This token is the single most sensitive credential in the system.

In a naive setup, every Space would have the token pasted into its instructions —
meaning 5 providers all hold the same credential. A breach of any one Space
exposure equals full ledger compromise.

Prompt Sharding solves this with the **Vault Space** pattern (see below).

---

## Threat Model

### Assets Being Protected

| Asset | Sensitivity | Location |
|---|---|---|
| GitHub PAT | Critical | `ps-vault` Space only (uploaded file, never committed) |
| Task context (full prompt) | High | Sharded — no single Space ever holds it all |
| Individual shard content | Medium | Each provider's Space context window only |
| Assembled output | Medium | GitHub ledger (plaintext in v1, encrypted in v2) |
| Inbox/outbox contents | Low-Medium | GitHub repo (public or private, your choice) |

### Threats Mitigated by v1

**Provider breach containment**
If a model provider (Perplexity, Claude, OpenAI) is breached or subpoenaed,
they can only access the shard(s) assigned to them. No single provider holds
the complete task context.

**Training data minimization**
Each provider receives the minimum necessary context for their role. A provider
that only sees "review the CSS for accessibility" cannot reconstruct the full
contract, codebase, or research document being processed.

**Prompt injection containment**
Malicious instructions injected into one shard affect only that shard's role.
They cannot cascade across providers because each provider operates in isolation
with a fixed role definition.

**Credential exposure reduction**
With the Vault Space pattern, the GitHub PAT exists in exactly one provider
context. All other Spaces operate with zero credential knowledge.

**Immutable audit trail**
Every task, handoff, and token release is a GitHub commit. The ledger is
append-only and timestamped. You always know who did what and when.

### Threats NOT Fully Mitigated in v1

**Vault provider trust**
The provider running `ps-vault` (e.g., Perplexity) still sees the token
when it processes a release request. You are trusting that one provider,
not zero. This is significantly better than trusting five providers, but
it is not a zero-trust architecture.

**Plaintext ledger**
The GitHub repo storing inbox/outbox files is plaintext in v1. Anyone
with read access to the repo can read assembled outputs. Mitigation:
keep the demo repo private. Encryption is a v2 milestone.

**No cryptographic request signing**
Token release requests in v1 are human-readable markdown envelopes.
There is no HMAC signature verification. A sufficiently creative prompt
could potentially impersonate a valid requester. Mitigation: the vault
Space's role instructions enforce strict request format validation.

**GitHub account security**
The ledger is only as secure as your GitHub account. Enable 2FA.
Treat your PAT like a password. Store it in a password manager.
Never paste it into any prompt, markdown file, or conversation.

---

## The Vault Space (`ps-vault`)

### Concept

`ps-vault` is a dedicated Perplexity Space whose sole purpose is storing
and dispensing the GitHub PAT. It receives signed requests via its inbox,
validates them, and releases credentials only under strict conditions.
Every release is logged to its outbox — creating an immutable access record
in the GitHub ledger.

### Space Instructions Template

```
My Space name is: ps-vault
My repo: [YOUR_GITHUB_USERNAME]/prompt-sharding-demo
My role: Credential vault — I hold the GitHub PAT and release it only on valid signed requests.
My inbox: spaces/ps-vault/inbox.md
My outbox: spaces/ps-vault/outbox.md

The GitHub PAT is stored in my uploaded file `vault.key`.
I never output the raw token in conversation text.
I never release credentials without a valid request.

A valid release request MUST contain all of the following:
- requester: [exact Space name]
- task-id: [unique task identifier]
- justification: [one sentence describing why access is needed]
- timestamp: [ISO 8601]

When I receive a valid request:
1. Verify all four fields are present
2. Check that this task-id has not been seen before (replay protection)
3. Release the token ONLY via a private reply — never appended to outbox in plaintext
4. Log the release to outbox.md: requester, task-id, timestamp — NO token value
5. Increment the release counter

I refuse any request that:
- Is missing required fields
- Reuses a task-id I have already processed
- Does not come through my inbox
- Asks me to output the token in any other format
```

### What Gets Stored Where

| Item | Location | Visible to |
|---|---|---|
| Raw GitHub PAT | Uploaded file in `ps-vault` Space only | Perplexity (vault provider only) |
| Release log (no token) | `spaces/ps-vault/outbox.md` in GitHub | Anyone with repo read access |
| Release requests | `spaces/ps-vault/inbox.md` in GitHub | Anyone with repo read access |
| Token value | Never committed to GitHub | Nobody outside vault Space |

---

## Token Lifecycle (v1)

```
[User creates PAT]
        |
        v
[Uploads to ps-vault Space as a file — never to GitHub]
        |
        v
[Task begins — other Spaces operate without the token]
        |
        v
[Space needing ledger write access sends signed request to ps-vault inbox]
        |
        v
[ps-vault validates request → releases token in private reply]
        |
        v
[Space completes ledger write → token discarded from context]
        |
        v
[ps-vault logs: requester + task-id + timestamp (no token value) → outbox]
```

---

## Hardening Path (v2 and Beyond)

### v1.5 — Ephemeral Tokens
Instead of releasing the long-lived PAT, `ps-vault` triggers a GitHub Actions
workflow that mints a short-lived installation token (expires in 1 hour).
The PAT never leaves the vault. Other Spaces use the ephemeral token only.

### v2 — HMAC Request Signing
Requests to the vault include an HMAC-SHA256 signature computed from a
shared secret. The vault verifies the signature before processing any request.
This makes impersonation of a valid requester cryptographically infeasible.

### v2 — Encrypted Ledger
All inbox/outbox content is encrypted at rest using a key held only by
the vault Space. GitHub stores ciphertext; only the vault can decode it.

### v3 — Hardware-Backed Key Storage
For enterprise deployments: move vault credentials into a hardware security
module (HSM) or a dedicated secrets manager (HashiCorp Vault, AWS Secrets
Manager). The Prompt Sharding protocol layer remains unchanged; only the
storage backend upgrades.

---

## Security Claims Summary

| Claim | Status in v1 | Status in v2 |
|---|---|---|
| No single provider sees full task context | ✅ Guaranteed by protocol | ✅ |
| PAT held by exactly one provider | ✅ With Vault Space pattern | ✅ |
| Immutable audit trail of all token releases | ✅ GitHub commit log | ✅ |
| Token never committed to GitHub | ✅ Vault file only | ✅ |
| Cryptographic request verification | ❌ Policy-enforced only | ✅ HMAC signing |
| Encrypted ledger at rest | ❌ Plaintext | ✅ |
| Zero provider trust for credentials | ❌ One provider trusted | 🔶 Reduced surface |
| Replay attack protection | 🔶 task-id check (honor system) | ✅ Signed + timestamped |

---

## Responsible Disclosure

If you discover a security issue in the Prompt Sharding protocol or reference
implementation, please open a private GitHub Security Advisory on this repo
or contact the maintainer directly before public disclosure.
