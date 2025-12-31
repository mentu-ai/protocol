# Example Workflow

A complete example showing the Mentu protocol in action.

---

## Scenario

A customer reports a bug. An agent fixes it. A human approves.

---

## Step 1: Capture Observation

Human sees a customer report:

```bash
mentu capture "Customer reported checkout fails on empty cart" --kind observation
```

Output:
```json
{"id": "mem_a1b2c3d4", "body": "Customer reported checkout fails on empty cart"}
```

---

## Step 2: Create Commitment

Human creates a commitment from the observation:

```bash
mentu commit "Fix empty cart checkout bug" --source mem_a1b2c3d4 --tag bug --tag checkout
```

Output:
```json
{"id": "cmt_e5f6g7h8", "body": "Fix empty cart checkout bug", "source": "mem_a1b2c3d4"}
```

---

## Step 3: Agent Claims

An AI agent checks for work:

```bash
mentu status
```

Output:
```
Open Commitments:
  cmt_e5f6g7h8: Fix empty cart checkout bug [bug, checkout]
```

Agent claims the commitment:

```bash
mentu claim cmt_e5f6g7h8 --actor agent:claude
```

---

## Step 4: Agent Works

Agent fixes the bug in the codebase.

---

## Step 5: Agent Captures Evidence

Agent records what was done:

```bash
mentu capture "Fixed null check in cart.ts:42, added test in cart.test.ts" \
  --kind evidence \
  --actor agent:claude
```

Output:
```json
{"id": "mem_m3n4o5p6", "body": "Fixed null check in cart.ts:42, added test..."}
```

---

## Step 6: Agent Submits

Agent submits for review:

```bash
mentu submit cmt_e5f6g7h8 \
  --evidence mem_m3n4o5p6 \
  --summary "Fixed empty cart checkout bug" \
  --actor agent:claude
```

Commitment state: `in_review`

---

## Step 7: Human Reviews

Human checks the review queue:

```bash
mentu status --in-review
```

Output:
```
In Review:
  cmt_e5f6g7h8: Fix empty cart checkout bug
    Submitted by: agent:claude
    Evidence: Fixed null check in cart.ts:42, added test in cart.test.ts
```

Human verifies the fix works.

---

## Step 8: Human Approves

Human approves the submission:

```bash
mentu approve cmt_e5f6g7h8 --comment "Verified fix works"
```

Commitment state: `closed`

---

## Final Ledger

```jsonl
{"id":"mem_a1b2c3d4","op":"capture","actor":"human:rashid","body":"Customer reported checkout fails on empty cart"}
{"id":"cmt_e5f6g7h8","op":"commit","actor":"human:rashid","body":"Fix empty cart checkout bug","source":"mem_a1b2c3d4"}
{"id":"op_i9j0k1l2","op":"claim","actor":"agent:claude","commitment":"cmt_e5f6g7h8"}
{"id":"mem_m3n4o5p6","op":"capture","actor":"agent:claude","body":"Fixed null check in cart.ts:42, added test","kind":"evidence"}
{"id":"op_q7r8s9t0","op":"submit","actor":"agent:claude","commitment":"cmt_e5f6g7h8","evidence":"mem_m3n4o5p6"}
{"id":"op_u1v2w3x4","op":"approve","actor":"human:rashid","commitment":"cmt_e5f6g7h8"}
```

---

## Key Points

1. **Observation came first** — The commitment traces to a memory
2. **Agent claimed before working** — No collision with others
3. **Evidence was specific** — File and line number included
4. **Human approved** — Asynchronous review, agent didn't wait
5. **Everything recorded** — Full audit trail in the ledger

---

*This is the protocol. Observe. Commit. Claim. Work. Evidence. Close.*
