# Changelog

## v2.1 — 2026-06-13

### Added
- **OKF profile** ([spec/OKF.md](spec/OKF.md)): the Open Knowledge Format (Andrej Karpathy's *LLM Wiki* and Google Cloud's *Open Knowledge Format*) with the `x-mentu` profile, which maps portable knowledge bundles onto the protocol's signal graph. Identity (`ns:type.slug`), trust (`level`/`decay`), five typed relations (`extends`, `supersedes`, `reinforces`, `constrains`, `balances_with`) mapping to protocol edges, and machine-readable twins.
- **Projection**: non-mutating, lossless import of a corpus that already uses its own frontmatter schema, deriving a parallel OKF bundle without rewriting the source.

## v2.0 — 2026-04-02

### Added
- **EpistemicSignal**: Unified signal type replaces Memory + Commitment as separate objects
- **Merkle chain**: SHA-256 hash chaining from genesis (64 zeros). Every signal links to the one before it.
- **Trust computation**: Seven-weight mechanical model, three confidence values (asserted, effective, current), temporal decay
- **Execution algebra**: Ten composable primitives (Step, Formula, Pipeline, Parallel, Compound, Adversarial, Convergent, Temporal, Sentinel, Substrate)
- **Typed relations**: `cites`, `extends`, `contradicts`, `refines` between signals
- **Observation levels**: `explicit`, `deductive`, `inductive`, `contradiction`
- **Semantic context**: Entities, intent, and domain tags on any signal
- **Sync scope**: `local`, `anonymous`, `full`, `cloud` per signal
- **Epistemic trace**: Parent/child causal links with causal depth
- **Invariants spec**: Append-only, Merkle integrity, citation gate (K1), mechanical trust, read-before-act
- **Trust spec**: Full specification of the seven-weight model and confidence lifecycle
- **Execution spec**: Ten primitives, composition algebra, embedding principle
- **CHANGELOG**: This file

### Changed
- **Operations**: Twelve → nine. `link`, `dismiss`, `triage` become annotation kinds.
- **Signal envelope**: Now includes `hash`, `prevHash`, `workspace`, optional `semantic`, `trust`, `trace`, `relations`, `observationLevel`, `sourceIds`, `syncScope`
- **Genesis Key**: Promoted from Draft to v1.0 Stable. Added `trust` section for weight overrides.
- **Agent instructions**: Added citation requirements, epistemic awareness, trust guidance
- **Sample ledger**: Full v2.0 signals with real SHA-256 hash chain
- **Workflow example**: Shows trust computation, hash chain growth, citation

### Backward Compatibility
- v1.0 ledgers are valid v2.0 ledgers. Missing fields default to null/empty.
- v1.0 operations `link`, `dismiss`, `triage` accepted as `annotate` with corresponding kind.
- `hash` and `prevHash` must be computed on v1.0 import.

## v1.0 — 2025-12-31

- Initial release
- Twelve operations: capture, commit, claim, release, close, annotate, submit, approve, reopen, link, dismiss, triage
- Two objects: Memory, Commitment
- JSONL ledger format
- Genesis Key (Draft)
- Agent instruction template
