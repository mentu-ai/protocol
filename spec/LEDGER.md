# Ledger Format

**Version**: 2.0

---

## Overview

The ledger is an append-only, hash-chained sequence of signals stored in JSON Lines format.

**Location**: `.mentu/ledger.jsonl`

---

## JSON Lines Format

Each line is a complete JSON object representing one EpistemicSignal.

```jsonl
{"id":"mem_a1b2c3d4","op":"capture","ts":"2026-04-02T10:30:00Z","actor":"human:rashid","workspace":"my-project","hash":"a7f3...","prevHash":"0000...0000","payload":{"body":"Customer reported bug","kind":"observation"}}
{"id":"cmt_e5f6g7h8","op":"commit","ts":"2026-04-02T10:31:00Z","actor":"human:rashid","workspace":"my-project","hash":"b8e4...","prevHash":"a7f3...","payload":{"body":"Fix the bug","kind":"commitment","source":"mem_a1b2c3d4"}}
```

---

## Signal Schema

### Identity

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `id` | Yes | string | Unique identifier (`mem_`, `cmt_`, `op_`, `ann_` prefix + 8 hex chars) |
| `op` | Yes | string | Operation: `capture`, `commit`, `claim`, `release`, `close`, `submit`, `approve`, `reopen`, `annotate` |
| `ts` | Yes | string | ISO 8601 timestamp in UTC |
| `actor` | Yes | string | Who performed the operation |
| `workspace` | Yes | string | Workspace name |

### Merkle Chain

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `hash` | Yes | string | SHA-256 content hash of this signal |
| `prevHash` | Yes | string | SHA-256 hash of previous signal. Genesis: 64 zeros. |

### Sync

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `syncScope` | No | string | `local`, `anonymous`, `full`, `cloud`. Default: implementation-defined. |

### Payload

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `payload.body` | Yes | string | Content: observation, obligation, evidence, annotation |
| `payload.kind` | No | string | Signal kind: `observation`, `commitment`, `evidence`, `step_result`, `finding`, `learning`, `event`, `claim`, `submission`, `approval`, `verdict`, `reopen` |
| `payload.source` | Conditional | string | Source signal ID. Required for `commit`. |
| `payload.commitment` | Conditional | string | Commitment ID. Required for `claim`, `release`, `submit`, `approve`, `close`, `reopen`. |
| `payload.evidence` | Conditional | string | Evidence signal ID. Required for `close`, `submit`. |
| `payload.summary` | No | string | Summary text for `submit`. |
| `payload.reason` | No | string | Reason for `release`, `reopen`. |
| `payload.comment` | No | string | Comment for `approve`. |
| `payload.target` | Conditional | string | Target signal ID. Required for `annotate`. |
| `payload.tags` | No | string[] | Labels for filtering. |
| `payload.refs` | No | string[] | Related signal IDs. |
| `payload.meta` | No | object | Arbitrary metadata. |
| `payload.duplicate_of` | No | string | Commitment ID for duplicate closure. |

### Semantic Context

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `semantic.entities` | No | string[] | Named things: function names, file paths, concepts |
| `semantic.intent` | No | string | What this signal is about: `fix_bug`, `implement_feature`, `verify_build` |
| `semantic.domain` | No | string[] | Domain tags: `security`, `testing`, `performance` |

### Trust Metadata

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `trust.confidence` | No | number | Asserted confidence (0.0–1.0). Frozen at creation. |
| `trust.verification` | No | string | `unverified`, `machine_verified`, `human_verified` |
| `trust.chain` | No | string[] | Mechanical checks that produced the confidence score |
| `trust.decayHalfLifeDays` | No | integer | Decay half-life in days |

See [TRUST.md](./TRUST.md) for the trust computation model.

### Epistemic Trace

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `trace.parent` | No | string[] | IDs of signals this one derives from |
| `trace.children` | No | string[] | IDs of signals derived from this one (updated by later signals) |
| `trace.causalDepth` | No | integer | How many causal links from raw observation. Default: 0. |

### Relations

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `relations` | No | array | Typed relationships to other signals |
| `relations[].type` | Yes | string | `cites`, `extends`, `contradicts`, `refines` |
| `relations[].targetId` | Yes | string | Target signal ID |

### Observation Level

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `observationLevel` | No | string | `explicit`, `deductive`, `inductive`, `contradiction` |
| `sourceIds` | No | string[] | Premise signal IDs for this observation |

---

## Operation Schemas

### capture

Creates a signal from observation.

```json
{
  "id": "mem_a1b2c3d4",
  "op": "capture",
  "ts": "2026-04-02T10:30:00Z",
  "actor": "human:rashid",
  "workspace": "my-project",
  "hash": "...",
  "prevHash": "...",
  "payload": {
    "body": "Customer reported checkout bug",
    "kind": "observation"
  }
}
```

### commit

Creates a commitment from a source signal.

```json
{
  "id": "cmt_e5f6g7h8",
  "op": "commit",
  "ts": "2026-04-02T10:31:00Z",
  "actor": "human:rashid",
  "workspace": "my-project",
  "hash": "...",
  "prevHash": "...",
  "payload": {
    "body": "Fix checkout bug",
    "kind": "commitment",
    "source": "mem_a1b2c3d4",
    "tags": ["bug", "checkout"]
  }
}
```

### claim

Takes responsibility for a commitment.

```json
{
  "id": "op_i9j0k1l2",
  "op": "claim",
  "ts": "2026-04-02T10:32:00Z",
  "actor": "agent:claude",
  "workspace": "my-project",
  "hash": "...",
  "prevHash": "...",
  "payload": {
    "body": "Claiming checkout bug fix",
    "kind": "claim",
    "commitment": "cmt_e5f6g7h8"
  }
}
```

### release

Gives up responsibility.

```json
{
  "id": "op_xxxxxxxx",
  "op": "release",
  "ts": "2026-04-02T11:00:00Z",
  "actor": "agent:claude",
  "workspace": "my-project",
  "hash": "...",
  "prevHash": "...",
  "payload": {
    "body": "Blocked on external dependency",
    "kind": "release",
    "commitment": "cmt_e5f6g7h8",
    "reason": "Blocked on external dependency"
  }
}
```

### close

Resolves with evidence (direct path).

```json
{
  "id": "op_xxxxxxxx",
  "op": "close",
  "ts": "2026-04-02T12:00:00Z",
  "actor": "agent:claude",
  "workspace": "my-project",
  "hash": "...",
  "prevHash": "...",
  "payload": {
    "body": "Closing with evidence",
    "kind": "verdict",
    "commitment": "cmt_e5f6g7h8",
    "evidence": "mem_xyz789"
  }
}
```

### submit

Requests closure, enters review.

```json
{
  "id": "op_xxxxxxxx",
  "op": "submit",
  "ts": "2026-04-02T12:00:00Z",
  "actor": "agent:claude",
  "workspace": "my-project",
  "hash": "...",
  "prevHash": "...",
  "payload": {
    "body": "Fixed null check in payment.ts:42",
    "kind": "submission",
    "commitment": "cmt_e5f6g7h8",
    "evidence": "mem_xyz789",
    "summary": "Fixed checkout bug"
  }
}
```

### approve

Accepts submission.

```json
{
  "id": "op_xxxxxxxx",
  "op": "approve",
  "ts": "2026-04-02T13:00:00Z",
  "actor": "human:rashid",
  "workspace": "my-project",
  "hash": "...",
  "prevHash": "...",
  "payload": {
    "body": "Verified fix works",
    "kind": "approval",
    "commitment": "cmt_e5f6g7h8",
    "comment": "Verified fix works"
  }
}
```

### reopen

Rejects submission or disputes closure.

```json
{
  "id": "op_xxxxxxxx",
  "op": "reopen",
  "ts": "2026-04-02T13:00:00Z",
  "actor": "human:rashid",
  "workspace": "my-project",
  "hash": "...",
  "prevHash": "...",
  "payload": {
    "body": "Edge case not handled",
    "kind": "reopen",
    "commitment": "cmt_e5f6g7h8",
    "reason": "Edge case not handled"
  }
}
```

### annotate

Attaches note to any signal. Subsumes v1.0 `link`, `dismiss`, `triage`.

```json
{
  "id": "ann_xxxxxxxx",
  "op": "annotate",
  "ts": "2026-04-02T10:35:00Z",
  "actor": "human:rashid",
  "workspace": "my-project",
  "hash": "...",
  "prevHash": "...",
  "payload": {
    "body": "High priority",
    "kind": "priority",
    "target": "cmt_e5f6g7h8"
  }
}
```

---

## Hash Computation

The `hash` field is computed as follows:

1. Serialize the signal to JSON with **sorted keys** and **no whitespace**
2. **Exclude** the `hash` and `prevHash` fields from the serialization
3. Compute SHA-256 of the resulting bytes
4. Encode as lowercase hexadecimal (64 characters)

```python
import hashlib, json

def compute_hash(signal):
    obj = {k: v for k, v in signal.items() if k not in ("hash", "prevHash")}
    canonical = json.dumps(obj, sort_keys=True, separators=(",", ":"))
    return hashlib.sha256(canonical.encode("utf-8")).hexdigest()
```

The `prevHash` field is the `hash` of the immediately preceding signal in the ledger. For the first signal (genesis), `prevHash` is:

```
0000000000000000000000000000000000000000000000000000000000000000
```

---

## ID Format

IDs use the format: `{prefix}_{8hex}`

| Prefix | Signal Type |
|--------|-------------|
| `mem` | Memory (capture) |
| `cmt` | Commitment (commit) |
| `op` | Operation (claim, release, close, submit, approve, reopen) |
| `ann` | Annotation (annotate) |

Example: `mem_a1b2c3d4`, `cmt_e5f6g7h8`, `op_i9j0k1l2`, `ann_m3n4o5p6`

---

## Timestamp Format

All timestamps use ISO 8601 format in UTC:

```
2026-04-02T10:30:00Z
2026-04-02T10:30:00.123Z
```

**Note**: Timestamps are metadata. Append order is truth.

---

## Actor Format

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
  "payload": { "body": "Issue from GitHub", "kind": "observation" }
}
```

If `source_key` is present, it MUST be unique. Duplicate `source_key` operations are rejected.

---

## File Structure

```
.mentu/
├── ledger.jsonl    # The append-only, hash-chained ledger
├── config.yaml     # Workspace configuration
├── genesis.key     # Constitutional identity (optional)
├── AGENTS.md       # Instructions for AI agents
└── .lock           # Write lock (runtime)
```

---

## Backward Compatibility

v1.0 signals that lack v2.0 fields are valid. On import:

- `hash` and `prevHash` must be computed and the chain reconstructed
- `workspace` defaults to `"default"`
- All optional v2.0 fields default to `null` or `[]`
- v1.0 operations `link`, `dismiss`, `triage` map to `annotate` with the corresponding `kind`

---

*Append-only. Hash-chained. Truth by replay.*
