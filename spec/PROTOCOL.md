# The Mentu Protocol

**Version**: 2.0
**Status**: Stable

---

## Overview

Mentu is a commitment protocol. It defines how observations become obligations, how obligations become work, and how work becomes closure.

The protocol is:
- **Append-only, hash-chained** — Nothing edited or deleted. Every entry links to the one before it.
- **Replay-based** — State is computed, never stored
- **Evidence-required** — Closure requires proof
- **Mechanically trusted** — Trust is computed from observation, not self-reported

---

## The Three Rules

1. **Commitments trace to observations** — Every `commit` references a `capture`
2. **Closure requires evidence** — Every `close` or `approve` references proof
3. **Append-only, hash-chained** — The ledger only grows. Every entry carries the hash of the one before it.

---

## The Signal

One type. One envelope. Every operation produces an **EpistemicSignal**.

Memory and Commitment are not separate objects. They are `payload.kind` values on the same signal type. A capture with `kind: "observation"` is a memory. A commit with `kind: "commitment"` is a commitment. The signal is the unit.

```json
{
  "id": "mem_a1b2c3d4",
  "op": "capture",
  "ts": "2026-04-02T10:30:00Z",
  "actor": "human:rashid",
  "workspace": "my-project",
  "hash": "e3b0c44298fc1c14...",
  "prevHash": "0000000000000000000000000000000000000000000000000000000000000000",
  "payload": {
    "body": "Customer reported checkout bug",
    "kind": "observation"
  }
}
```

Full schema in [LEDGER.md](./LEDGER.md).

---

## The Nine Operations

### Core

| Operation | Effect |
|-----------|--------|
| `capture` | Record an observation. Creates a signal. |
| `commit` | Create an obligation. Must reference a source signal. |
| `claim` | Take responsibility for a commitment. |
| `release` | Give up responsibility. Returns to `open`. |
| `close` | Resolve with evidence. Direct path. |
| `annotate` | Attach a note to any signal. |

### Review

| Operation | Effect |
|-----------|--------|
| `submit` | Request closure. Enters `in_review`. |
| `approve` | Accept submission. Transitions to `closed`. |
| `reopen` | Reject or dispute. Returns to `claimed`. |

`link`, `dismiss`, and `triage` from v1.0 become annotation kinds — not protocol operations. Use `annotate` with `kind: "link"`, `kind: "dismiss"`, or `kind: "triage"`.

---

## State Machine

### Commitment States

```
                    ┌──────────────────────────────────┐
                    │                                  │
                    ▼                                  │
┌──────┐  claim  ┌─────────┐  submit  ┌───────────┐   │
│ open │────────▶│ claimed │─────────▶│ in_review │   │
└──────┘         └─────────┘          └───────────┘   │
                    ▲   │                 │    │       │
                    │   │ release         │    │       │
                    │   ▼                 │    │       │
                    │ (open)          approve  reopen  │
                    │                     │    │       │
                    │                     ▼    ▼       │
                    │              ┌────────┐ ┌────────┐
                    │              │ closed │ │reopened│
                    │              └────────┘ └───┬────┘
                    │                             │
                    └─────────────────────────────┘
                              (claim)
```

| State | Meaning |
|-------|---------|
| `open` | No owner, available for claim |
| `claimed` | Has owner, work in progress |
| `in_review` | Submitted, awaiting approval |
| `closed` | Resolved with evidence |

**Direct close**: `claimed` → `closed` still permitted for backwards compatibility.

**Review flow**: `claimed` → `in_review` → `closed` recommended for agent workflows.

---

## Merkle Chain

Every signal carries two hash fields:

| Field | Description |
|-------|-------------|
| `hash` | SHA-256 of this signal's content |
| `prevHash` | SHA-256 of the previous signal |

The genesis signal has `prevHash` of 64 zeros:

```
0000000000000000000000000000000000000000000000000000000000000000
```

Hash computation: SHA-256 of the canonical JSON representation (sorted keys, no whitespace) of the signal, **excluding** `hash` and `prevHash` from the input. See [LEDGER.md](./LEDGER.md) for the algorithm.

A conforming implementation MUST verify the chain on read. If `signal[n].prevHash != signal[n-1].hash`, the chain is broken.

---

## Relations

Signals can declare typed relationships to other signals:

| Relation | Meaning |
|----------|---------|
| `cites` | This signal uses another as evidence |
| `extends` | This signal builds on another |
| `contradicts` | This signal disagrees with another |
| `refines` | This signal improves on another |

Relations are append-only. A relation from A to B does not modify B.

---

## Observation Levels

Every signal can declare how it was derived:

| Level | Meaning |
|-------|---------|
| `explicit` | Direct observation or raw fact |
| `deductive` | Follows necessarily from prior signals |
| `inductive` | Follows probably from prior signals |
| `contradiction` | Conflicts with prior signals |

Observation level enables provenance chain traversal — you can ask "show me everything derived from this observation" or "show me all contradictions."

---

## Semantic Context

Optional structured metadata on any signal:

| Field | Type | Description |
|-------|------|-------------|
| `entities` | string[] | Named things (functions, files, concepts) |
| `intent` | string | What this signal is about |
| `domain` | string[] | Classification tags (security, testing, etc.) |

Enables contradiction detection and causal reasoning across the ledger.

---

## Sync Scope

Each signal declares its sync boundary:

| Scope | Meaning |
|-------|---------|
| `local` | Never leaves this machine |
| `anonymous` | Syncs without actor identity |
| `full` | Syncs with full metadata |
| `cloud` | Syncs to remote service |

Default sync scope depends on signal kind. Implementation-defined.

---

## Trust

Signals carry mechanical trust metadata. Trust is computed from observation — exit codes, test results, duration, context utilization — not from self-report.

Three confidence values:

| Value | When Set | Mutable |
|-------|----------|---------|
| `asserted_confidence` | Creation | Never |
| `effective_confidence` | Recomputed | On citation, contradiction, reinforcement |
| `current_confidence` | Read time | Never stored |

Full trust specification in [TRUST.md](./TRUST.md).

---

## Epistemic Trace

Signals form causal chains:

| Field | Type | Description |
|-------|------|-------------|
| `parent` | string[] | IDs of signals this one derives from |
| `children` | string[] | IDs of signals derived from this one |
| `causalDepth` | integer | How many links deep from raw observation |

The citation gate invariant: signals of kind `finding`, `step_result`, or `learning` MUST have at least one parent. Evidence must cite its source.

---

## The Accountability Airlock

The `in_review` state is the accountability airlock.

**Why it exists:**
- Agents complete work and submit
- Validation runs automatically
- Humans review asynchronously
- Agents don't wait for humans
- Commitments wait in `in_review`

**Tiered review:**

| Tier | Automated | Human | Use Case |
|------|-----------|-------|----------|
| Tier 1 | Full validation | None | Routine work, high trust |
| Tier 2 | Full + async review | Can reopen within window | Medium risk |
| Tier 3 | Full + sync gate | Must approve before close | High risk |

Tier classification is defined in [Genesis Key](./GENESIS.md).

---

## Operation Envelope

Every operation follows this structure:

```json
{
  "id": "op_xxxxxxxx",
  "op": "capture",
  "ts": "2026-04-02T10:30:00Z",
  "actor": "human:rashid",
  "workspace": "my-project",
  "hash": "...",
  "prevHash": "...",
  "payload": { ... }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Unique identifier (`mem_`, `cmt_`, `op_`, `ann_` prefix) |
| `op` | Yes | Operation type |
| `ts` | Yes | ISO 8601 timestamp (UTC) |
| `actor` | Yes | Who performed the operation |
| `workspace` | Yes | Workspace name |
| `hash` | Yes | SHA-256 content hash |
| `prevHash` | Yes | SHA-256 of previous signal |
| `payload` | Yes | Operation-specific data |
| `syncScope` | No | Sync boundary (default: implementation-defined) |
| `semantic` | No | Entities, intent, domain tags |
| `trust` | No | Mechanical trust metadata |
| `trace` | No | Parent/child causal links |
| `relations` | No | Typed relationships to other signals |
| `observationLevel` | No | How this signal was derived |
| `sourceIds` | No | Premise signal IDs for observation |
| `source_key` | No | Idempotency key for deduplication |

---

## Invariants

### Structural
- `commit` payload MUST reference existing signal as source
- `close` payload MUST reference evidence
- `claim` payload MUST reference existing commitment

### State
- Cannot `claim` a closed commitment
- Cannot `claim` a commitment claimed by another actor
- Cannot `close` without evidence
- Cannot `release` if not owner

### Integrity
- IDs are globally unique within workspace
- Operations are ordered by append sequence, not timestamp
- Ledger is append-only
- Merkle chain is continuous — `signal[n].prevHash == signal[n-1].hash`

### Epistemic
- Signals of kind `finding`, `step_result`, `learning` MUST have `trace.parent` (citation gate K1)
- Trust scores MUST be computed mechanically, never self-reported

See [INVARIANTS.md](./INVARIANTS.md) for the full invariant specification.

---

## State Computation

State is computed by replaying operations. Never stored.

```python
def compute_state(ledger, commitment_id):
    state = {"state": "open", "owner": None, "evidence": None}

    for signal in ledger:
        target = signal.payload.get("commitment")
        if target != commitment_id:
            continue

        match signal.op:
            case "claim":
                state["state"] = "claimed"
                state["owner"] = signal.actor
            case "release":
                state["state"] = "open"
                state["owner"] = None
            case "submit":
                state["state"] = "in_review"
            case "approve":
                state["state"] = "closed"
                state["evidence"] = state.get("submitted_evidence")
            case "reopen":
                state["state"] = "claimed"
            case "close":
                state["state"] = "closed"
                state["evidence"] = signal.payload.get("evidence")

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
| `E_CHAIN_BROKEN` | Merkle chain integrity violation |
| `E_CITATION_REQUIRED` | Signal kind requires trace.parent |

---

## Backward Compatibility

v1.0 ledgers are valid v2.0 ledgers. Missing fields default to null:

| v2.0 Field | Default when absent |
|------------|---------------------|
| `hash` | Implementation must compute on import |
| `prevHash` | Implementation must reconstruct chain |
| `workspace` | `"default"` |
| `syncScope` | `"local"` |
| `semantic` | `null` |
| `trust` | `null` |
| `trace` | `null` |
| `relations` | `[]` |
| `observationLevel` | `null` |
| `sourceIds` | `[]` |

v1.0 operations `link`, `dismiss`, `triage` are accepted as `annotate` with the corresponding `kind`.

---

## Conformance

A conforming v2.0 implementation MUST:

1. Store ledger as append-only JSONL at `.mentu/ledger.jsonl`
2. Implement all nine operations
3. Enforce all structural, state, and integrity invariants
4. Compute state by replay
5. Maintain Merkle chain integrity
6. Return specified error codes
7. Enforce citation gate (K1) on applicable signal kinds

A conforming implementation SHOULD:
1. Compute mechanical trust scores
2. Support semantic context and relations
3. Support sync scope configuration

Two implementations are compatible if:
- Given identical ledger, they compute identical state
- Given identical operation, they produce identical validation result
- Given identical signal content, they compute identical hash

---

## The Sacred Invariant

Every module follows one rule:

**Read ledger, write ops.**

No component stores its own state. State is always computed by replaying the ledger.

---

*Nine operations. One signal type. Three rules.*
