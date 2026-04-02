# Mentu

**The Commitment Protocol**

A substrate for accountable action. Observations become commitments. Commitments require evidence. Evidence is hash-chained and trust-scored. Everything is recorded. Nothing is edited.

---

## The Open Protocol

The protocol is open. The format is open. Run it yourself.

If the substrate isn't open — inspectable, auditable, forkable — we've failed before we started.

---

## What Mentu Is

A commitment ledger. Append-only. Hash-chained. Three rules:

1. **Commitments trace to observations** — Every obligation has an origin
2. **Closure requires evidence** — Proving done, not marking done
3. **Append-only, hash-chained** — Nothing edited, nothing deleted. Every entry carries the hash of the one before it.

The ledger is the truth. State is computed by replay.

---

## One Signal Type

**EpistemicSignal** — the universal entry. Everything in the ledger is a signal.

A capture with `kind: "observation"` is a memory. A commit with `kind: "commitment"` is a commitment. Evidence, findings, approvals — all signals. One type. One schema. One chain.

```json
{
  "id": "mem_a1b2c3d4",
  "op": "capture",
  "ts": "2026-04-02T10:30:00Z",
  "actor": "human:rashid",
  "workspace": "my-project",
  "hash": "bed4c1c8...",
  "prevHash": "00000000...",
  "payload": { "body": "Customer reported checkout bug", "kind": "observation" },
  "trust": { "confidence": 0.85, "verification": "machine_verified" }
}
```

---

## The Nine Operations

### Core
| Operation | Effect |
|-----------|--------|
| `capture` | Record an observation → Signal |
| `commit` | Create an obligation → Signal |
| `claim` | Take responsibility |
| `release` | Give up responsibility |
| `close` | Resolve with evidence |
| `annotate` | Attach a note |

### Review
| Operation | Effect |
|-----------|--------|
| `submit` | Request closure, enter `in_review` |
| `approve` | Accept submission → `closed` |
| `reopen` | Reject, return to `claimed` |

---

## Trust

Trust is computed, never self-reported. Seven mechanical observations — exit code, test pass rate, context utilization, completion signal, duration, error state, evidence depth — weighted into a confidence score.

Three confidence values:
- **Asserted** — frozen at creation
- **Effective** — evolves as the graph grows (citations increase it, contradictions decrease it)
- **Current** — effective × temporal decay (evidence fades over time)

See [Trust Computation](./spec/TRUST.md).

---

## The Execution Algebra

Ten composable primitives for orchestrating agent work:

| Primitive | Purpose |
|-----------|---------|
| Step | One agent, one intent |
| Formula | Ordered step sequence |
| Pipeline | Sequential formula chain |
| Parallel | Concurrent in isolation |
| Compound | Dependency graph |
| Adversarial | Red/Blue self-correction |
| Convergent | N strategies, best wins |
| Temporal | Scheduled execution |
| Sentinel | Continuous monitoring |
| Substrate | Infrastructure tuning |

They form a closed algebra — any primitive can embed any other. A step runs identically alone or inside a 200-step compound.

See [Execution Algebra](./spec/EXECUTION.md).

---

## What `mentu init` Creates

```
.mentu/
├── ledger.jsonl      # The append-only, hash-chained log
├── config.yaml       # Workspace configuration
├── AGENTS.md         # Instructions for AI agents
└── genesis.key       # Constitutional identity (optional)
```

Four files. The ledger is the truth. The config is how it connects. The AGENTS.md is how agents learn to behave. The key is who governs.

---

## The Agent Protocol

How do agents know what to do?

**They read `.mentu/AGENTS.md`.**

Every AI agent — Claude, GPT, Cursor, Devin, any future agent — starts by reading that file. It tells them: check for work, claim before starting, capture evidence, submit for review.

The agent doesn't need special integration. It needs to read a file and run commands.

```bash
mentu status                          # Check for work
mentu claim cmt_def456                # Take responsibility
# ... do the work ...
mentu capture "Fixed bug" --kind evidence   # Record evidence
mentu submit cmt_def456 --evidence mem_ghi789  # Submit for review
```

This works for any agent. No SDK required. Just a terminal.

---

## The Sacred Invariant

Every module follows one rule:

**Read ledger, write ops.**

No component stores its own state. State is always computed by replaying the ledger.

---

## Documentation

- [Protocol Specification](./spec/PROTOCOL.md) — The nine operations, state machine, Merkle chain, invariants
- [Ledger Format](./spec/LEDGER.md) — Signal schema, hash computation, JSON Lines format
- [Trust Computation](./spec/TRUST.md) — Seven-weight model, three confidence values, temporal decay
- [Execution Algebra](./spec/EXECUTION.md) — Ten composable primitives, composition algebra
- [Invariants](./spec/INVARIANTS.md) — Five structural guarantees
- [Genesis Key](./spec/GENESIS.md) — Workspace constitution, permissions, trust weights
- [Agent Instructions](./agents/AGENTS.md) — Template for teaching agents
- [Examples](./examples/) — Sample ledger with real hashes, end-to-end workflow

---

## The Principle

Mentu is not a system agents integrate with.

**Mentu is a protocol agents learn to follow.**

The difference matters. Integration requires code. Protocol requires reading a file and running commands.

Any agent that can read and execute shell commands can use Mentu. Today. Without waiting for SDKs or plugins.

---

## License

MIT — Free as in freedom.

---

*A Merkle-chained ledger where commitments require evidence.*
