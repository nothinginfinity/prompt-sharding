# Shard Schema

The `config/shards.json` file defines how a task is split into shards and which agent receives each one.

## Schema

```typescript
interface ShardConfig {
  version: string;           // schema version, e.g. "0.1"
  taskTypes: TaskShardMap;   // map of task type → shard routing rules
}

interface TaskShardMap {
  [taskType: string]: ShardRoute[];
}

interface ShardRoute {
  shardIndex: number;        // 1-based index of this shard
  role: string;              // human-readable role label
  agent: string;             // agent space name, e.g. "alice.mmcp"
  contextSlice: ContextSlice; // what part of the input this agent sees
  outputLabel: string;       // label for this agent’s output in the receipt
  required: boolean;         // if true, task fails if this shard fails
}

type ContextSlice =
  | { type: "full" }                          // agent sees everything (single-agent fallback)
  | { type: "fields"; fields: string[] }      // agent sees specific fields only
  | { type: "lines"; start: number; end: number } // agent sees a line range
  | { type: "role-prompt"; prompt: string };  // agent sees only a role-specific prompt
```

## Example: Contract Review

```json
{
  "version": "0.1",
  "taskTypes": {
    "contract-review": [
      {
        "shardIndex": 1,
        "role": "structural-analysis",
        "agent": "alice.mmcp",
        "contextSlice": {
          "type": "role-prompt",
          "prompt": "You are reviewing the structure and completeness of this contract. Do not assess risk or summarize. Output: list of sections present, missing, and structurally ambiguous."
        },
        "outputLabel": "structure",
        "required": true
      },
      {
        "shardIndex": 2,
        "role": "risk-identification",
        "agent": "bob.mmcp",
        "contextSlice": {
          "type": "role-prompt",
          "prompt": "You are a legal risk analyst. Identify clauses that pose risk to the party requesting this review. Do not summarize or assess structure. Output: list of risky clauses with explanation."
        },
        "outputLabel": "risks",
        "required": true
      },
      {
        "shardIndex": 3,
        "role": "plain-language-summary",
        "agent": "carol.mmcp",
        "contextSlice": {
          "type": "role-prompt",
          "prompt": "Summarize this contract in plain language for a non-lawyer. Do not assess risk or structure. Output: 3-5 sentence summary."
        },
        "outputLabel": "summary",
        "required": false
      }
    ],
    "code-review": [
      {
        "shardIndex": 1,
        "role": "architecture-review",
        "agent": "alice.mmcp",
        "contextSlice": { "type": "role-prompt", "prompt": "Review only the architecture and design patterns. Do not review security or tests." },
        "outputLabel": "architecture",
        "required": true
      },
      {
        "shardIndex": 2,
        "role": "security-review",
        "agent": "bob.mmcp",
        "contextSlice": { "type": "role-prompt", "prompt": "Review only for security vulnerabilities. Do not review architecture or tests." },
        "outputLabel": "security",
        "required": true
      },
      {
        "shardIndex": 3,
        "role": "test-coverage",
        "agent": "carol.mmcp",
        "contextSlice": { "type": "role-prompt", "prompt": "Review only test coverage and test quality. Do not review architecture or security." },
        "outputLabel": "tests",
        "required": false
      }
    ],
    "game-turn": [
      {
        "shardIndex": 1,
        "role": "game-engine",
        "agent": "game-engine.mmcp",
        "contextSlice": { "type": "full" },
        "outputLabel": "new-state",
        "required": true
      }
    ]
  }
}
```

## Shard Design Rules

1. **Minimum necessary context** — each shard’s `role-prompt` should describe only what that agent needs to see
2. **No cross-shard references** — shards must not point to each other’s content
3. **Role isolation** — roles should not semantically overlap
4. **Stateless agents** — agents process their shard independently, no shared memory
5. **Signed outputs** — each agent signs its output envelope before the router assembles the receipt
