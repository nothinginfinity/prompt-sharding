# Quick Start

A working two-agent prompt-sharding setup using two Perplexity Spaces and a GitHub repo as the ledger. No server required — runs from an iPhone.

## What you’ll build

```
You (iPhone)
  → write a task to the ledger
    → router splits it into shards
      → Alice (Perplexity Space 1) receives Shard 1
      → Bob   (Perplexity Space 2) receives Shard 2
        → both write outputs to their outboxes
          → router assembles receipt
            → you read the result
```

## Step 1 — Create the ledger repo

Create a new GitHub repo (public or private). This is your ledger.

Add this structure:
```
my-ledger/
├── NETWORK.md          (copy from docs/NETWORK-template.md)
├── config/
│   └── shards.json       (copy from docs/shard-schema.md example)
└── spaces/
    ├── alice.mmcp/
    │   ├── inbox.md
    │   └── outbox.md
    └── bob.mmcp/
        ├── inbox.md
        └── outbox.md
```

## Step 2 — Create two Perplexity Spaces

**Alice’s Space instructions:**
```
My space name: alice.mmcp
My repo: <your-username>/<your-ledger-repo>
My inbox: spaces/alice.mmcp/inbox.md
My outbox: spaces/alice.mmcp/outbox.md
To reach Bob: append to spaces/bob.mmcp/inbox.md

When I receive a shard envelope in my inbox, I process only my assigned
role and write a signed output envelope to my outbox. I never read
other agents’ inboxes or attempt to reconstruct the full task.
```

**Bob’s Space instructions:**
```
My space name: bob.mmcp
My repo: <your-username>/<your-ledger-repo>
My inbox: spaces/bob.mmcp/inbox.md
My outbox: spaces/bob.mmcp/outbox.md
To reach Alice: append to spaces/alice.mmcp/inbox.md

When I receive a shard envelope in my inbox, I process only my assigned
role and write a signed output envelope to my outbox. I never read
other agents’ inboxes or attempt to reconstruct the full task.
```

## Step 3 — Install the router

Add [`mmcp-inbox-router`](https://github.com/nothinginfinity/mmcp-inbox-router) as a GitHub Actions workflow in your ledger repo. The router will:
- Scan inboxes on every push
- Classify envelopes
- Route shards per `config/shards.json`
- Assemble receipts when all shard outputs are present

## Step 4 — Send your first task

Append a task envelope to `spaces/alice.mmcp/inbox.md`:

```json
{
  "id": "task-001",
  "taskId": "task-001",
  "shardIndex": 1,
  "shardOf": 2,
  "from": "router.mmcp",
  "to": "alice.mmcp",
  "role": "structural-analysis",
  "payload": {
    "subject": "Shard 1/2 — structural analysis",
    "content": "Review the structure of this document: [your content here]",
    "contentType": "text/plain"
  },
  "sentAt": "2026-04-28T15:00:00Z",
  "signature": "signed:router.mmcp:task-001-shard-1",
  "policy": "no-cross-shard-read,sign-output"
}
```

The router fires on the push, Alice processes her shard, Bob processes his. You read the assembled receipt.

## That’s it.

Two phones. Two Spaces. One ledger. No server. Full prompt sharding.
