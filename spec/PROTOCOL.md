# The Mentu Protocol

**Version**: 1.0
**Status**: Stable

---

## Overview

Mentu is a commitment protocol. It defines how observations become obligations, how obligations become work, and how work becomes closure.

The protocol is:
- **Append-only** — Nothing is edited or deleted
- **Replay-based** — State is computed, never stored
- **Evidence-required** — Closure requires proof

---

## The Three Rules

1. **Commitments trace to observations** — Every `commit` references a `capture`
2. **Closure requires evidence** — Every `close` references a proof memory
3. **Append-only** — The ledger only grows

---

## The Two Objects

### Memory

Something observed. Created by `capture`. Immutable.

```json
{
  "id": "mem_a1b2c3d4",
  "body": "Customer reported checkout bug",
  "kind": "observation",
  "ts": "2025-01-15T10:30:00Z"
}
```

### Commitment

Something owed. Created by `commit`. Requires a source memory. Requires evidence to close.

```json
{
  "id": "cmt_e5f6g7h8",
  "body": "Fix checkout bug",
  "source": "mem_a1b2c3d4",
  "state": "open",
  "owner": null
}
```

---

## The Twelve Operations

### Core Operations

| Operation | Input | Effect |
|-----------|-------|--------|
| `capture` | body, kind? | Creates Memory |
| `commit` | body, source | Creates Commitment (state: `open`) |
| `claim` | commitment | Sets owner, state → `claimed` |
| `release` | commitment | Clears owner, state → `open` |
| `close` | commitment, evidence | state → `closed` |
| `annotate` | target, body | Attaches note to record |

### Review Operations

| Operation | Input | Effect |
|-----------|-------|--------|
| `submit` | commitment, summary | state → `in_review` |
| `approve` | commitment | state → `closed` |
| `reopen` | commitment, reason | state → `claimed` |

### Triage Operations

| Operation | Input | Effect |
|-----------|-------|--------|
| `link` | source, target | Connects to commitment |
| `dismiss` | memory, reason | Marks not actionable |
| `triage` | memories[], summary | Records triage session |

---

## State Machine

### Commitment States

```
         claim                submit               approve
  open ───────► claimed ───────► in_review ───────► closed
    ▲              │                  │
    │   release    │                  │ reopen
    └──────────────┘                  │
                   ◄──────────────────┘
```

| State | Meaning |
|-------|---------|
| `open` | No owner, available for claim |
| `claimed` | Has owner, work in progress |
| `in_review` | Submitted, awaiting approval |
| `closed` | Resolved with evidence |

### Memory States

| State | Meaning |
|-------|---------|
| `untriaged` | Not yet processed |
| `linked` | Connected to a commitment |
| `dismissed` | Marked as not actionable |
| `committed` | Is source of a commitment |

---

## Operation Envelope

Every operation follows this structure:

```json
{
  "id": "op_xxxxxxxx",
  "op": "capture",
  "ts": "2025-01-15T10:30:00Z",
  "actor": "human:rashid",
  "payload": { ... }
}
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Unique identifier (`op_`, `mem_`, `cmt_` prefix) |
| `op` | Yes | Operation type |
| `ts` | Yes | ISO 8601 timestamp |
| `actor` | Yes | Who performed the operation |
| `payload` | Yes | Operation-specific data |
| `source_key` | No | Idempotency key for deduplication |

---

## Invariants

### Structural
- `commit.source` MUST reference existing memory
- `close.evidence` MUST reference existing memory
- `claim.commitment` MUST reference existing commitment

### State
- Cannot `claim` a closed commitment
- Cannot `claim` a commitment claimed by another actor
- Cannot `close` without evidence
- Cannot `release` if not owner

### Integrity
- IDs are globally unique within workspace
- Operations are ordered by append sequence, not timestamp
- Ledger is append-only

---

## State Computation

State is computed by replaying operations. Never stored.

```python
def compute_commitment_state(ledger, commitment_id):
    state = {"state": "open", "owner": None, "evidence": None}

    for op in ledger:
        if op.payload.commitment != commitment_id:
            continue

        if op.op == "claim":
            state["state"] = "claimed"
            state["owner"] = op.actor
        elif op.op == "release":
            state["state"] = "open"
            state["owner"] = None
        elif op.op == "submit":
            state["state"] = "in_review"
        elif op.op == "approve":
            state["state"] = "closed"
            state["evidence"] = state.get("submitted_evidence")
        elif op.op == "reopen":
            state["state"] = "claimed"
        elif op.op == "close":
            state["state"] = "closed"
            state["evidence"] = op.payload.evidence

    return state
```

---

## Error Codes

| Code | Meaning |
|------|---------|
| `E_INVALID_OP` | Unknown operation type |
| `E_MISSING_FIELD` | Required field absent |
| `E_REF_NOT_FOUND` | Referenced ID does not exist |
| `E_ALREADY_CLOSED` | Commitment already closed |
| `E_NOT_OWNER` | Actor is not current owner |
| `E_ALREADY_CLAIMED` | Commitment claimed by another |
| `E_DUPLICATE_ID` | ID already exists |
| `E_PERMISSION_DENIED` | Genesis Key forbids operation |
| `E_CONSTRAINT_VIOLATED` | Constraint not satisfied |

---

## Conformance

A conforming implementation MUST:

1. Store ledger as append-only JSONL at `.mentu/ledger.jsonl`
2. Implement all twelve operations
3. Enforce all invariants
4. Compute state by replay
5. Return specified error codes

Two implementations are compatible if:
- Given identical ledger, they compute identical state
- Given identical operation, they produce identical validation result

---

*Twelve operations. One file. Three rules.*
