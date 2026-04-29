# Security Model — Prompt Sharding

> Version: 0.2 | Last updated: 2026-04-28

This document defines what Prompt Sharding protects, what it does not protect,
the honest threat model for v1, and the architectural path toward stronger guarantees.

---

## The Two Core Security Primitives

Prompt Sharding's security model rests on two compounding properties that,
combined, produce guarantees no single-provider AI workflow can match:

### 1. Stateless Sessions

Providers are **ephemeral compute**, not memory stores. A Space holds context
only for the duration of an active conversation. Once output is committed to
the MMCP ledger (your GitHub repo), the conversation is deleted. The provider
retains nothing you haven't chosen to keep.

Resuming any task requires exactly two things:
- The Space Instructions (role, rules, inbox/outbox paths — held by you)
- One prompt: *"Check your inbox"*

The ledger is the persistent memory. The provider is interchangeable compute.

### 2. Context Sharding

No single provider ever receives the complete picture. Each provider receives
only the shard assigned to its role — the minimum context necessary to complete
that role's task. Even if a provider retains server-side infrastructure logs,
those logs contain only a fragment. The complete task context exists nowhere
except the assembled ledger — which you own.

**The combination:** stateless sessions bound the *time window* of exposure.
Context sharding bounds the *scope* of exposure. Together they make any single
provider's data both temporary and incomplete.

---

## Threat Model

### Assets Being Protected

| Asset | Sensitivity | Location |
|---|---|---|
| GitHub PAT | Critical | `ps-vault` Space only — never committed to GitHub |
| Full task context | High | Assembled ledger only — no provider ever holds it all |
| Individual shard content | Medium | One provider's context window, one session only |
| Assembled output | Medium | GitHub ledger (plaintext v1, encrypted v2) |
| Inbox/outbox contents | Low–Medium | GitHub repo (public or private, your choice) |
| Provider inference logs | Low | Partial shard only — incomplete without other shards |

---

## Threats Mitigated

### Provider breach containment
If a model provider is breached or subpoenaed, they can only produce:
- The shard(s) assigned to their role
- Infrastructure logs from sessions that have since been deleted

Neither reveals the complete task. The attacker would need to simultaneously
breach every provider used in the workflow to reconstruct the full context —
and even then, only for sessions that overlapped in their retention window.

### Training data minimization
Each provider receives a role-specific fragment. A provider that only saw
"review the CSS for accessibility" cannot reconstruct the contract, codebase,
or research document being processed. Deleting the conversation after each
session removes it from the provider's conversation history entirely. What
remains in infrastructure logs is a fragment, not a document.

### Provider log sharding
This is the compounding property. Even if every major provider retains
infrastructure logs for 30 days:

```
Provider A log:  [shard 1 only — structure]      → useless alone
Provider B log:  [shard 2 only — risk analysis]  → useless alone
Provider C log:  [shard 3 only — summary]        → useless alone

Complete picture: exists only in YOUR ledger
```

An adversary would need to breach three separate providers, correlate logs
across them, and reassemble the shards — all within the retention window.
This is not a theoretical attack; it is an expensive, coordinated one.

### Prompt injection containment
Malicious instructions injected into one shard affect only that role.
They cannot cascade because each provider operates in a fixed-role context
with no awareness of the other shards.

### Credential exposure reduction
With the Vault Space pattern, the GitHub PAT exists in exactly one provider
context for one session. All other Spaces operate with zero credential knowledge.
Every token release is logged to the ledger with requester, task-id, and
timestamp — but never the token value itself.

### Provider lock-in elimination
Context lives in your ledger, not in any provider's conversation history.
Switching providers mid-project requires no migration. The inbox is the
context. Any provider with the Space Instructions can resume from exactly
where the last session ended.

### Immutable audit trail
Every task, handoff, token release, and assembled output is a GitHub commit.
The ledger is append-only and timestamped. You always know who did what,
when, and in what order.

---

## Stateless Session Pattern

This is the operational security practice that activates the full threat model.

```
[Task begins]
        |
        v
[Open Space in provider — context loaded from inbox]
        |
        v
[Complete task — output committed to outbox in GitHub ledger]
        |
        v
[Delete conversation from provider UI]
        |
        v
[Provider holds: nothing in conversation history]
[Provider may hold: infrastructure log of the session]
[Infrastructure log contains: one shard only — incomplete]
        |
        v
[Resume anytime, any provider: Space Instructions + "check your inbox"]
```

### Why deleting the conversation matters
Most providers distinguish between:
- **Conversation history** — user-visible, used for memory and context, explicitly
  stored and potentially used in training pipelines
- **Infrastructure logs** — server-side request/response logs, retained for
  operational reasons, separate from the product experience

Deleting the conversation eliminates the first category entirely. The second
category is bounded by the provider's retention policy (typically 30–90 days)
and contains only the shard — not the full context.

### The honest framing
This is not zero-knowledge. It is **bounded, sharded, temporary exposure**
versus the default of **unbounded, complete, permanent storage**. That is a
fundamental improvement in the threat model, not a marginal one.

---

## The Vault Space (`ps-vault`)

### Concept

`ps-vault` is a dedicated Space whose sole purpose is storing and dispensing
the GitHub PAT. It receives structured requests via its inbox, validates them,
and releases credentials only under strict conditions. Every release is logged
to its outbox — an immutable access record in the GitHub ledger.

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
| Raw GitHub PAT | Uploaded file in `ps-vault` Space only | Vault provider only, one session at a time |
| Release log (no token) | `spaces/ps-vault/outbox.md` in GitHub | Anyone with repo read access |
| Release requests | `spaces/ps-vault/inbox.md` in GitHub | Anyone with repo read access |
| Token value | Never committed to GitHub | Nobody outside vault session |

---

## Security Claims Summary

| Claim | v1 Status | v2 Status |
|---|---|---|
| No single provider sees full task context | ✅ Protocol guarantee | ✅ |
| Provider log exposure is sharded and partial | ✅ Structural guarantee | ✅ |
| Conversation history deletable after each session | ✅ Operational practice | ✅ |
| Provider is interchangeable — no lock-in | ✅ Ledger owns all state | ✅ |
| PAT held by exactly one provider, one session | ✅ Vault Space pattern | ✅ |
| Token never committed to GitHub | ✅ Vault file only | ✅ |
| Immutable audit trail of all token releases | ✅ GitHub commit log | ✅ |
| Cryptographic request verification | ❌ Policy-enforced only | ✅ HMAC-SHA256 |
| Encrypted ledger at rest | ❌ Plaintext | ✅ |
| Ephemeral tokens (never release long-lived PAT) | ❌ | ✅ GitHub Actions TTL |
| Zero provider trust for credentials | ❌ One provider trusted | 🔶 Reduced surface |
| Infrastructure log elimination | ❌ Bounded by retention policy | 🔶 Minimize via session length |

---

## Hardening Path

### v1.5 — Vault Space + Stateless Sessions (policy-enforced)
Delete every conversation after output is committed. Vault holds PAT in
uploaded file. All other Spaces operate credential-free. Provider exposure
is sharded, temporary, and incomplete.

### v2 — Ephemeral Tokens + HMAC Signing
Vault mints short-lived GitHub Actions tokens (1-hour TTL) instead of
releasing the long-lived PAT. Request signing via HMAC-SHA256 makes
impersonation of a valid requester cryptographically infeasible.

### v2 — Encrypted Ledger
All inbox/outbox content is encrypted at rest. GitHub stores ciphertext.
Only the vault Space can decrypt. Even with full repo read access, an
attacker sees encrypted fragments.

### v3 — Hardware-Backed Key Storage
For enterprise deployments: vault credentials move into an HSM or dedicated
secrets manager (HashiCorp Vault, AWS Secrets Manager). The Prompt Sharding
protocol layer is unchanged — only the storage backend upgrades.

---

## Responsible Disclosure

If you discover a security issue in the Prompt Sharding protocol or reference
implementation, please open a private GitHub Security Advisory on this repo
or contact the maintainer directly before public disclosure.
