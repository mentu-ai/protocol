# Ledger Format

**Version**: 1.0

---

## Overview

The ledger is an append-only sequence of operations stored in JSON Lines format.

**Location**: `.mentu/ledger.jsonl`

---

## JSON Lines Format

Each line is a complete JSON object representing one operation.

```jsonl
{"id":"op_a1b2c3d4","op":"capture","ts":"2025-01-15T10:30:00Z","actor":"human:rashid","body":"Customer reported bug","kind":"observation"}
{"id":"cmt_e5f6g7h8","op":"commit","ts":"2025-01-15T10:31:00Z","actor":"human:rashid","body":"Fix the bug","source":"mem_a1b2c3d4"}
{"id":"op_i9j0k1l2","op":"claim","ts":"2025-01-15T10:32:00Z","actor":"agent:claude","commitment":"cmt_e5f6g7h8"}
```

---

## Operation Schemas

### capture

Creates a memory.

```json
{
  "id": "mem_xxxxxxxx",
  "op": "capture",
  "ts": "2025-01-15T10:30:00Z",
  "actor": "human:rashid",
  "body": "Customer reported checkout bug",
  "kind": "observation",
  "path": "docs/RESULT-xxx.md",
  "refs": ["cmt_abc12345"],
  "meta": {}
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `body` | Yes | Content of the observation |
| `kind` | No | Classification (observation, task, evidence, document, etc.) |
| `path` | No | Document path for kind=document |
| `refs` | No | Array of related IDs |
| `meta` | No | Arbitrary metadata object |

### commit

Creates a commitment from a source memory.

```json
{
  "id": "cmt_xxxxxxxx",
  "op": "commit",
  "ts": "2025-01-15T10:31:00Z",
  "actor": "human:rashid",
  "body": "Fix checkout bug",
  "source": "mem_a1b2c3d4",
  "tags": ["bug", "checkout"],
  "meta": {}
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `body` | Yes | Description of obligation |
| `source` | Yes | Memory ID that originated this |
| `tags` | No | Labels for filtering |
| `meta` | No | Arbitrary metadata |

### claim

Takes responsibility for a commitment.

```json
{
  "id": "op_xxxxxxxx",
  "op": "claim",
  "ts": "2025-01-15T10:32:00Z",
  "actor": "agent:claude",
  "commitment": "cmt_e5f6g7h8"
}
```

### release

Gives up responsibility.

```json
{
  "id": "op_xxxxxxxx",
  "op": "release",
  "ts": "2025-01-15T11:00:00Z",
  "actor": "agent:claude",
  "commitment": "cmt_e5f6g7h8",
  "reason": "Blocked on external dependency"
}
```

### close

Resolves with evidence.

```json
{
  "id": "op_xxxxxxxx",
  "op": "close",
  "ts": "2025-01-15T12:00:00Z",
  "actor": "agent:claude",
  "commitment": "cmt_e5f6g7h8",
  "evidence": "mem_xyz789"
}
```

Or closes as duplicate:

```json
{
  "id": "op_xxxxxxxx",
  "op": "close",
  "ts": "2025-01-15T12:00:00Z",
  "actor": "agent:claude",
  "commitment": "cmt_e5f6g7h8",
  "duplicate_of": "cmt_other123"
}
```

### submit

Requests closure, enters review.

```json
{
  "id": "op_xxxxxxxx",
  "op": "submit",
  "ts": "2025-01-15T12:00:00Z",
  "actor": "agent:claude",
  "commitment": "cmt_e5f6g7h8",
  "evidence": "mem_xyz789",
  "summary": "Fixed null check in payment.ts:42"
}
```

### approve

Accepts submission.

```json
{
  "id": "op_xxxxxxxx",
  "op": "approve",
  "ts": "2025-01-15T13:00:00Z",
  "actor": "human:rashid",
  "commitment": "cmt_e5f6g7h8",
  "comment": "Verified fix works"
}
```

### reopen

Rejects submission or disputes closure.

```json
{
  "id": "op_xxxxxxxx",
  "op": "reopen",
  "ts": "2025-01-15T13:00:00Z",
  "actor": "human:rashid",
  "commitment": "cmt_e5f6g7h8",
  "reason": "Edge case not handled"
}
```

### annotate

Attaches note to any record.

```json
{
  "id": "op_xxxxxxxx",
  "op": "annotate",
  "ts": "2025-01-15T10:35:00Z",
  "actor": "human:rashid",
  "target": "cmt_e5f6g7h8",
  "body": "High priority",
  "kind": "priority"
}
```

### link

Connects memory or commitment to a commitment.

```json
{
  "id": "op_xxxxxxxx",
  "op": "link",
  "ts": "2025-01-15T10:36:00Z",
  "actor": "human:rashid",
  "source": "mem_related",
  "target": "cmt_e5f6g7h8",
  "kind": "related"
}
```

### dismiss

Marks memory as not actionable.

```json
{
  "id": "op_xxxxxxxx",
  "op": "dismiss",
  "ts": "2025-01-15T10:37:00Z",
  "actor": "human:rashid",
  "memory": "mem_a1b2c3d4",
  "reason": "Already fixed in v1.2"
}
```

### triage

Records a triage session.

```json
{
  "id": "op_xxxxxxxx",
  "op": "triage",
  "ts": "2025-01-15T10:40:00Z",
  "actor": "human:rashid",
  "reviewed": ["mem_1", "mem_2", "mem_3"],
  "summary": "Processed morning inbox"
}
```

---

## ID Format

IDs use the format: `{prefix}_{8hex}`

| Prefix | Object |
|--------|--------|
| `mem` | Memory |
| `cmt` | Commitment |
| `op` | Operation |

Example: `mem_a1b2c3d4`, `cmt_e5f6g7h8`, `op_i9j0k1l2`

---

## Timestamp Format

All timestamps use ISO 8601 format in UTC:

```
2025-01-15T10:30:00Z
2025-01-15T10:30:00.123Z
```

**Note**: Timestamps are metadata. Append order is truth.

---

## Actor Format

Actors identify who performed the operation:

| Format | Example |
|--------|---------|
| Human | `human:rashid` |
| Agent | `agent:claude` |
| System | `system:sync` |
| Email | `rashid@example.com` |

---

## Idempotency

The `source_key` field enables safe replay:

```json
{
  "id": "op_xxxxxxxx",
  "op": "capture",
  "source_key": "github:issue:123",
  "body": "Issue from GitHub"
}
```

If `source_key` is present, it MUST be unique. Duplicate `source_key` operations are rejected.

---

## File Structure

```
.mentu/
├── ledger.jsonl    # The append-only ledger
├── config.yaml     # Workspace configuration
├── genesis.key     # Constitutional identity (optional)
├── AGENTS.md       # Instructions for AI agents
└── .lock           # Write lock (runtime)
```

---

*Append-only. JSON Lines. Truth by replay.*
