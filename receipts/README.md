# Receipts

Assembled task receipts are written here by the router when all required shard outputs are present.

## Format

Each receipt is a markdown file named `<taskId>.md`:

```markdown
# Receipt — <taskId>

**Task type:** contract-review
**Assembled at:** <ISO 8601>
**Shards:** 3/3 complete

## structure (alice.mmcp)

<alice’s output>

## risks (bob.mmcp)

<bob’s output>

## summary (carol.mmcp)

<carol’s output>
```
