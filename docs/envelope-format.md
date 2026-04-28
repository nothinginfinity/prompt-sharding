# Shard Envelope Format

Every shard is delivered to an agent as a signed envelope in the agent’s `inbox.md`.

## Format

```json
{
  "id": "shard-<taskId>-<shardIndex>",
  "taskId": "<globally unique task identifier>",
  "shardIndex": 1,
  "shardOf": 3,
  "from": "router.mmcp",
  "to": "alice.mmcp",
  "role": "structural-analysis",
  "payload": {
    "subject": "Shard 1/3 — structural analysis",
    "content": "<shard content — never the full task>",
    "contentType": "text/plain"
  },
  "sentAt": "<ISO 8601>",
  "signature": "signed:router.mmcp:<shard-id>",
  "policy": "no-cross-shard-read,no-full-context-reconstruction,sign-output"
}
```

## Fields

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique shard ID: `shard-<taskId>-<shardIndex>` |
| `taskId` | ✅ | Task this shard belongs to |
| `shardIndex` | ✅ | 1-based position of this shard |
| `shardOf` | ✅ | Total number of shards in this task |
| `from` | ✅ | Always `router.mmcp` for shard delivery |
| `to` | ✅ | Target agent space name |
| `role` | ✅ | Human-readable role label from `shards.json` |
| `payload.content` | ✅ | The shard content — never the full task |
| `signature` | ✅ | Router signature for authenticity |
| `policy` | ✅ | Comma-separated policy constraints |

## Policy Values

| Value | Meaning |
|---|---|
| `no-cross-shard-read` | Agent must not attempt to read other agents’ inboxes |
| `no-full-context-reconstruction` | Agent must not attempt to reassemble the full task |
| `sign-output` | Agent must sign its output envelope before writing to outbox |
| `no-persist` | Agent must not store this shard beyond the current session |
| `audit-log` | Router will log this shard delivery to the audit trail |

## Agent Output Envelope

When an agent completes its shard, it writes a signed output envelope to its `outbox.md`:

```json
{
  "id": "output-<shardId>",
  "taskId": "<taskId>",
  "shardIndex": 1,
  "from": "alice.mmcp",
  "to": "router.mmcp",
  "role": "structural-analysis",
  "payload": {
    "subject": "Output: structural analysis complete",
    "content": "<agent output>",
    "contentType": "text/plain"
  },
  "sentAt": "<ISO 8601>",
  "signature": "signed:alice.mmcp:output-<shardId>"
}
```

The router reads all agent outboxes, verifies signatures, and assembles the final receipt.
