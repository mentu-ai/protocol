# Mentu

**The Commitment Protocol**

A substrate for accountable action. Observations become commitments. Commitments require evidence. Evidence enables closure. Everything is recorded.

---

## The Parallel

Git is free. GitHub makes billions.

**Mentu is free. [mentu.ai](https://mentu.ai) is the business.**

The protocol is open. The format is open. Run it yourself.

---

## Why Open Source

We're building a substrate for accountable action.

If the substrate itself isn't open — inspectable, auditable, forkable — we've failed before we started.

**The medium must embody the message.**

A substrate must be:
- **Trustable** — You can't build accountability infrastructure that isn't itself accountable
- **Ubiquitous** — Git won because anyone could use it, anywhere, for anything, no permission needed
- **Scientific** — Researchers need access to inspect, extend, validate. Closed substrates don't become foundations.

---

## What Mentu Is

A commitment ledger. Append-only. Three rules:

1. **Commitments trace to observations** — Every obligation has an origin
2. **Closure requires evidence** — Proving done, not marking done
3. **Append-only** — Nothing edited, nothing deleted

The ledger is the truth. State is computed by replay.

---

## The Innovation

**Commitment as a first-class primitive that crosses system boundaries.**

That's the conceptual innovation. The technology to implement it is boring:

- Postgres (constraints, indexes, locking)
- HTTP API (point queries)
- Simple schema (one object, clear guarantees)

The product is the abstraction, not the infrastructure.

---

## What `mentu init` Creates

```
.mentu/
├── ledger.jsonl      # The append-only log
├── config.yaml       # Workspace configuration
├── AGENTS.md         # Instructions for AI agents
└── genesis.key       # Constitutional identity (optional)
```

Four files. The ledger is the truth. The config is how it connects. The AGENTS.md is how agents learn to behave. The key is who governs.

---

## The Twelve Operations

### Core (v0.1)
| Operation | Effect |
|-----------|--------|
| `capture` | Record an observation → Memory |
| `commit` | Create an obligation → Commitment |
| `claim` | Take responsibility |
| `release` | Give up responsibility |
| `close` | Resolve with evidence |
| `annotate` | Attach a note |

### Review (v1.0)
| Operation | Effect |
|-----------|--------|
| `submit` | Request closure, enter `in_review` |
| `approve` | Accept submission → `closed` |
| `reopen` | Reject, return to `claimed` |

### Triage (v0.8)
| Operation | Effect |
|-----------|--------|
| `link` | Connect memory/commitment to commitment |
| `dismiss` | Mark memory as not actionable |
| `triage` | Record a triage session |

---

## The Agent Protocol

How do agents know what to do?

**They read `.mentu/AGENTS.md`.**

Every AI agent — Claude, GPT, Cursor, Devin, any future agent — starts by reading that file. It tells them:

1. What Mentu is
2. How to check for work
3. How to claim work
4. How to produce evidence
5. How to close work
6. What they cannot do

The agent doesn't need special integration. It needs to read a file and run commands.

### Agent Workflow

```bash
# 1. Read instructions
cat .mentu/AGENTS.md

# 2. Check for work
mentu status

# 3. Claim before starting
mentu claim <commitment>

# 4. Do the work
# ...

# 5. Capture evidence
mentu capture "What I did" --kind evidence

# 6. Close with proof
mentu close <commitment> --evidence <memory>
```

This works for any agent. No SDK required. Just a terminal.

---

## The Sacred Invariant

Every module follows one rule:

**Read ledger, write ops.**

No component stores its own state. State is always computed by replaying the ledger.

---

## Quickstart

```bash
# Install the CLI
npm install -g mentu

# Initialize a workspace
mentu init

# Observe something
mentu capture "Customer reported bug in checkout"

# Create a commitment
mentu commit "Fix checkout bug" --source mem_abc123

# Take responsibility
mentu claim cmt_def456

# Do the work...

# Capture evidence
mentu capture "Fixed null check in payment.ts:42" --kind evidence

# Close with proof
mentu close cmt_def456 --evidence mem_ghi789
```

---

## Ecosystem

| Component | Purpose | Repo |
|-----------|---------|------|
| **Protocol** | The specification | [mentu-ai/mentu](https://github.com/mentu-ai/mentu) |
| **CLI** | Node.js implementation | [mentu-ai/mentu-cli](https://github.com/mentu-ai/mentu-cli) |
| **Plugin** | Claude Code integration | [mentu-ai/mentu-plugin](https://github.com/mentu-ai/mentu-plugin) |
| **Proxy** | Sync service | [mentu-ai/mentu-proxy](https://github.com/mentu-ai/mentu-proxy) |
| **Service** | mentu.ai platform | Private |

---

## Documentation

- [Protocol Specification](./spec/PROTOCOL.md) — The twelve operations, state machines, invariants
- [Ledger Format](./spec/LEDGER.md) — JSONL structure, operation envelope
- [Genesis Key](./spec/GENESIS.md) — Workspace constitution, permissions, constraints
- [Agent Instructions](./agents/AGENTS.md) — Template for teaching agents
- [Examples](./examples/) — Sample ledgers and workflows

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

*A ledger where commitments require evidence.*
