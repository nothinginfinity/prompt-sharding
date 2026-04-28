# Roadmap

## v0.1 — Specification ✅
- Full protocol specification (this repo)
- Shard schema defined
- Envelope format defined
- Threat model documented
- Enterprise guide written
- Quick start guide written

## v0.2 — Router Shard Dispatch
- `shards.json` loader in `mmcp-inbox-router`
- Shard splitting logic: takes a full task, produces N shard envelopes
- Shard delivery: writes each envelope to the correct agent inbox
- Output assembly: when all shard outputs present, assembles receipt
- New intent type: `task:shard-complete`

## v0.3 — Signed Envelope Enforcement
- Router verifies agent output signatures before assembly
- Invalid signatures reject the shard output and flag for review
- Audit log entry written for every shard delivery and output
- `sign-output` policy enforced at the router level

## v0.4 — stone-vault Integration
- Shard content encrypted before writing to ledger
- Only router holds decryption keys (via stone-vault)
- Ledger stores ciphertext — ledger breach no longer yields plaintext shards
- `no-persist` policy communicated to agents via envelope metadata

## v0.5 — Cross-Provider Test Suite
- End-to-end test: Perplexity + Claude + ChatGPT on a real document
- Breach simulation: verify shard isolation holds under ledger read access
- Correlation resistance test: verify shard outputs are not individually meaningful
- Performance baseline: measure latency per shard for async workflows

## v1.0 — Production Hardening
- GitHub Enterprise integration guide with reference Actions workflow
- HIPAA compliance checklist
- SOC 2 readiness notes
- Security audit by external reviewer
- Public launch
