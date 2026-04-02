# Protocol Invariants

**Version**: 2.0
**Status**: Stable

---

## Overview

Five invariants govern every operation on the ledger. Break any one and the system's guarantees fail.

---

## 1. Append-Only

Signals are never updated or deleted.

Once written, a signal exists forever. Corrections are new signals — a `contradicts` relation, not an edit. Retractions are new signals — an `annotate` with `kind: "retraction"`, not a deletion.

This is what makes the ledger an audit trail. If entries can change, history is negotiable.

---

## 2. Merkle Integrity

Every signal's `prevHash` equals the prior signal's `hash`.

The chain starts at genesis (64 zeros) and grows with every append. A conforming implementation MUST verify the chain on read. If any link is broken, the ledger is compromised.

```
signal[0].prevHash = "0000...0000"
signal[1].prevHash = signal[0].hash
signal[2].prevHash = signal[1].hash
...
signal[n].prevHash = signal[n-1].hash
```

The hash excludes `hash` and `prevHash` from its own computation. See [LEDGER.md](./LEDGER.md) for the algorithm.

---

## 3. Citation Gate (K1)

Signals of kind `finding`, `step_result`, or `learning` MUST have at least one entry in `trace.parent`.

Evidence must cite its source. A finding without provenance is an assertion, not evidence. An implementation MUST reject signals of these kinds that lack a parent reference.

**Why:** Without citation, the graph has no edges. Without edges, there is no provenance traversal, no contradiction detection, no trust propagation. The citation gate is what makes the ledger epistemically useful, not just a log.

---

## 4. Mechanical Trust

Trust scores derive from observation, not self-report.

An agent cannot declare `confidence: 0.95` on its own output. The seven-weight model computes confidence from exit codes, test results, context utilization, completion signals, duration, and evidence depth. See [TRUST.md](./TRUST.md).

**Why:** Self-reported trust is meaningless. A confident hallucination scores itself highly. Mechanical trust removes the agent from its own evaluation.

---

## 5. Read-Before-Act (RECOMMENDED)

Query prior evidence before acting.

An agent without epistemic context is epistemically blind — it may contradict existing evidence, duplicate completed work, or ignore known constraints. This invariant is RECOMMENDED, not REQUIRED. Enforcement is implementation-specific.

**Why:** The value of the ledger scales with how much it is read. A ledger that is only written to is a log. A ledger that is read before every action is a substrate.

---

## Enforcement

| Invariant | Level | Failure |
|-----------|-------|---------|
| Append-only | MUST | Implementation rejects mutations |
| Merkle integrity | MUST | `E_CHAIN_BROKEN` on verification failure |
| Citation gate (K1) | MUST | `E_CITATION_REQUIRED` on applicable kinds |
| Mechanical trust | MUST | Implementation computes trust, rejects self-reported values |
| Read-before-act | SHOULD | Implementation logs warning on blind writes |

---

*Five rules. The ledger is only as strong as its weakest invariant.*
