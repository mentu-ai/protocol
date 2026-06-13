# The Commitment Protocol

Observations become obligations. Obligations require evidence. Evidence is hash-chained, trust-scored, and append-only. State is never stored — always computed by replaying the ledger.

```jsonl
{"id":"mem_a1b2c3d4","op":"capture","hash":"bed4c1c8...","prevHash":"0000...","payload":{"body":"Customer reported checkout fails on empty cart","kind":"observation"}}
{"id":"cmt_e5f6g7h8","op":"commit","hash":"56789c80...","prevHash":"bed4c1c8...","payload":{"body":"Fix empty cart checkout bug","source":"mem_a1b2c3d4"}}
{"id":"op_i9j0k1l2","op":"claim","hash":"c491a41c...","prevHash":"56789c80...","payload":{"commitment":"cmt_e5f6g7h8"}}
{"id":"mem_m3n4o5p6","op":"capture","hash":"9b280af3...","prevHash":"c491a41c...","payload":{"body":"Fixed null check in cart.ts:42","kind":"evidence"},"trust":{"confidence":0.85}}
{"id":"op_q7r8s9t0","op":"submit","hash":"815fd202...","prevHash":"9b280af3...","payload":{"commitment":"cmt_e5f6g7h8","evidence":"mem_m3n4o5p6"}}
{"id":"op_u1v2w3x4","op":"approve","hash":"7ddb5ab4...","prevHash":"815fd202...","payload":{"commitment":"cmt_e5f6g7h8"},"trust":{"confidence":0.95,"verification":"human_verified"}}
```

Six signals. One unbroken hash chain. Observation → obligation → claim → evidence → submission → approval. That's the whole protocol.

---

## Three Rules

1. **Commitments trace to observations.** Every obligation has an origin.
2. **Closure requires evidence.** Proving done, not marking done.
3. **Append-only, hash-chained.** Nothing edited. Nothing deleted. Every entry carries the hash of the one before it.

---

## Who This Is For

| You are | Start here |
|---------|------------|
| **Building an agent framework** | [Protocol Spec](./spec/PROTOCOL.md) — nine operations, state machine, Merkle chain |
| **Implementing a ledger** | [Ledger Format](./spec/LEDGER.md) — signal schema, hash computation algorithm |
| **Adding trust scoring** | [Trust Spec](./spec/TRUST.md) — seven-weight model, three confidence values, decay |
| **Orchestrating multi-step work** | [Execution Algebra](./spec/EXECUTION.md) — ten composable primitives |
| **Teaching an agent the protocol** | [Agent Instructions](./agents/AGENTS.md) — drop-in template for any AI agent |
| **Exploring by example** | [Sample Ledger](./examples/sample-ledger.jsonl) — six signals with real SHA-256 hashes |

---

## One Signal Type

Everything in the ledger is an **EpistemicSignal**. Observations, commitments, evidence, approvals — one type, one schema, one chain.

```json
{
  "id": "mem_a1b2c3d4",
  "op": "capture",
  "ts": "2026-04-02T10:30:00Z",
  "actor": "human:rashid",
  "workspace": "my-project",
  "hash": "bed4c1c85f6561da...",
  "prevHash": "0000000000000000...",
  "payload": {
    "body": "Customer reported checkout fails on empty cart",
    "kind": "observation"
  },
  "semantic": { "entities": ["checkout"], "domain": ["bug"] },
  "trust": { "confidence": 0.85, "verification": "machine_verified" },
  "trace": { "parent": [], "causalDepth": 0 },
  "relations": [],
  "observationLevel": "explicit"
}
```

---

## Nine Operations

| Operation | What it does |
|-----------|-------------|
| `capture` | Record an observation |
| `commit` | Create an obligation from an observation |
| `claim` | Take responsibility |
| `release` | Give it back |
| `close` | Resolve with evidence |
| `submit` | Request closure — enters `in_review` |
| `approve` | Accept a submission |
| `reopen` | Reject — back to `claimed` |
| `annotate` | Attach a note to any signal |

State machine:

```
  open ──claim──▶ claimed ──submit──▶ in_review ──approve──▶ closed
    ▲                │                     │
    └──release───────┘                     │
    └──────────────────────reopen──────────┘
```

---

## Mechanical Trust

Trust is computed from observation, never self-reported. Seven weighted signals — exit code, test pass rate, context utilization, completion detection, duration, error state, evidence depth — produce a confidence score between 0 and 1.

Three values track how trust evolves:

| Value | Lifecycle |
|-------|-----------|
| **Asserted** | Frozen at creation. What the mechanical model computed. |
| **Effective** | Evolves. Citations increase it. Contradictions decrease it. |
| **Current** | Read-time only. Effective × temporal decay. Never stored. |

Evidence fades: `2^(−age_days / half_life_days)`. A 90-day-old signal at half-life retains 50% confidence. Fresh evidence always weighs more.

[Full trust specification →](./spec/TRUST.md)

---

## Execution Algebra

Ten composable primitives. Each one accepts an intent, produces evidence, and records both to the ledger.

| Primitive | Signature |
|-----------|-----------|
| **Step** | `S: (Intent, Context) → (Evidence, Trust)` |
| **Formula** | `fold(S₁, S₂, …, Sₙ)` — ordered steps, knowledge accumulates |
| **Pipeline** | `F₁ ; F₂ ; … ; Fₙ` — sequential formulas, conditional routing |
| **Parallel** | `‖{F₁, F₂, …, Fₙ}` — concurrent, isolated |
| **Compound** | `G = (V, E)` — dependency graph, topological execution |
| **Adversarial** | `A(F_blue, F_red)` — Blue defends, Red attacks, trust adjusts |
| **Convergent** | `C({F₁…Fₙ}, σ)` — N strategies, selector picks the winner |
| **Temporal** | Scheduled execution with evidence TTL |
| **Sentinel** | Continuous monitoring with progressive escalation |
| **Substrate** | Meta-operations on trust weights and configuration |

They form a closed algebra — any primitive embeds any other without modification. A step runs identically alone or inside a 200-step compound.

[Full execution specification →](./spec/EXECUTION.md)

---

## Merkle Chain

Every signal carries the SHA-256 hash of the one before it. The genesis signal links to 64 zeros. The chain is verifiable end-to-end.

```python
import hashlib, json

def compute_hash(signal):
    obj = {k: v for k, v in signal.items() if k not in ("hash", "prevHash")}
    canonical = json.dumps(obj, sort_keys=True, separators=(",", ":"))
    return hashlib.sha256(canonical.encode("utf-8")).hexdigest()
```

The [sample ledger](./examples/sample-ledger.jsonl) ships with real hashes you can verify.

---

## Agent Workflow

Any agent that can read a file and run shell commands can follow the protocol. No SDK. No integration. Drop [AGENTS.md](./agents/AGENTS.md) into `.mentu/` and the agent knows what to do.

```bash
mentu status                                    # read the ledger
mentu claim cmt_e5f6g7h8                        # take responsibility
# ... do the work ...
mentu capture "Fixed null check" --kind evidence  # record what happened
mentu submit cmt_e5f6g7h8 --evidence mem_xyz789   # submit for review
```

Works for Claude, GPT, Cursor, Devin, Codex — any agent, today.

---

## Five Invariants

1. **Append-only.** Signals are never updated or deleted.
2. **Merkle integrity.** Every `prevHash` matches the prior `hash`.
3. **Citation gate.** Findings must reference their source.
4. **Mechanical trust.** Computed from observation, never self-reported.
5. **Read before act.** Query prior evidence before acting. *(recommended)*

[Full invariant specification →](./spec/INVARIANTS.md)

---

## Workspace

```
.mentu/
├── ledger.jsonl    # append-only, hash-chained
├── config.yaml     # workspace configuration
├── AGENTS.md       # agent instruction template
└── genesis.key     # constitutional identity (optional)
```

The ledger is the source of truth. State is always computed by replaying it. Nothing else stores state.

---

## Specifications

| Spec | What it covers |
|------|---------------|
| [PROTOCOL.md](./spec/PROTOCOL.md) | Nine operations, state machine, signal envelope, conformance |
| [LEDGER.md](./spec/LEDGER.md) | EpistemicSignal schema, hash algorithm, JSON Lines format |
| [TRUST.md](./spec/TRUST.md) | Seven-weight model, three confidences, temporal decay |
| [EXECUTION.md](./spec/EXECUTION.md) | Ten primitives, composition algebra, embedding principle |
| [INVARIANTS.md](./spec/INVARIANTS.md) | Five structural guarantees |
| [GENESIS.md](./spec/GENESIS.md) | Workspace constitution, permissions, trust weights |
| [OKF.md](./spec/OKF.md) | The Open Knowledge Format and the x-mentu profile: portable knowledge bundles on the protocol's signal graph |

---

## Why Open

A substrate for accountable action that isn't itself accountable is a contradiction. The protocol is MIT-licensed, inspectable, auditable, and forkable. If you can improve it, do.

---

## License

[MIT](./LICENSE)

---

*A Merkle-chained ledger where commitments require evidence.*
