# Genesis Key

**Version**: 1.0
**Status**: Draft

---

## Overview

The Genesis Key is an optional workspace constitution. It defines who owns the workspace, who can do what, and what requires approval.

**Location**: `.mentu/genesis.key`
**Format**: YAML

---

## When to Use

**Without Genesis Key**: Anyone can do anything. No permissions. No constraints. Good for personal projects.

**With Genesis Key**: Boundaries become constitutional. Actors have roles. Operations have requirements. Good for teams.

---

## Schema

```yaml
# Identity
identity:
  name: "Acme Corp Engineering"
  owner: "human:rashid"
  created: "2025-01-15"

# Actors
actors:
  - id: "human:rashid"
    role: admin
  - id: "human:alice"
    role: contributor
  - id: "agent:claude"
    role: agent
  - id: "agent:*"
    role: agent

# Permissions
permissions:
  admin:
    - "*"
  contributor:
    - capture
    - commit
    - claim
    - release
    - close
    - annotate
    - submit
  agent:
    - capture
    - claim
    - release
    - submit
    - annotate
  reviewer:
    - approve
    - reopen

# Constraints
constraints:
  claim:
    require_claim: true  # Must claim before closing
  close:
    require_evidence: true  # Must have evidence

# Validation tiers
validation:
  tiers:
    tier_1:
      auto_close: true
      validators: [technical]
    tier_2:
      auto_close: true
      review_window: 24h
      validators: [technical, safety]
    tier_3:
      auto_close: false
      require_human: true
      validators: [technical, safety, intent]

  classification:
    - match: { tags: [routine] }
      tier: tier_1
    - match: { tags: [new-pattern, migration] }
      tier: tier_2
    - match: { tags: [security, financial, production] }
      tier: tier_3
    - default: tier_2

# Federation
federation:
  enabled: false
  sync_target: "https://mentu.ai"
  workspaces: []
```

---

## Sections

### identity

Who owns this workspace.

```yaml
identity:
  name: "Project Name"
  owner: "human:rashid"
  created: "2025-01-15"
  description: "A brief description"
```

### actors

Who can operate in this workspace.

```yaml
actors:
  - id: "human:rashid"
    role: admin
  - id: "agent:claude"
    role: agent
  - id: "agent:*"         # Wildcard for any agent
    role: agent
  - id: "*"               # Wildcard for anyone
    role: contributor
```

### permissions

What each role can do.

```yaml
permissions:
  admin:
    - "*"                 # All operations
  contributor:
    - capture
    - commit
    - claim
    - release
    - close
    - annotate
  agent:
    - capture
    - claim
    - release
    - submit             # Agents submit, don't close directly
    - annotate
  reviewer:
    - approve
    - reopen
```

### constraints

Rules that must be satisfied.

```yaml
constraints:
  # Require claim before any close
  claim:
    require_claim: true

  # Evidence is mandatory
  close:
    require_evidence: true

  # Tags required for commits
  commit:
    require_tags: true

  # Specific tag constraints
  security:
    require_approval: true
    approvers: [human:*]
```

### validation

Tiered review requirements.

```yaml
validation:
  tiers:
    tier_1:
      auto_close: true
      validators: [technical]

    tier_2:
      auto_close: true
      review_window: 24h
      validators: [technical, safety]

    tier_3:
      auto_close: false
      require_human: true
      validators: [technical, safety, intent]

  classification:
    - match: { tags: [routine] }
      tier: tier_1
    - match: { tags: [database, external-api] }
      tier: tier_2
    - match: { tags: [security, financial] }
      tier: tier_3
    - default: tier_2
```

### federation

Multi-workspace coordination.

```yaml
federation:
  enabled: true
  sync_target: "https://mentu.ai"
  workspaces:
    - "acme-corp/frontend"
    - "acme-corp/backend"
    - "acme-corp/mobile"
```

---

## Enforcement

When Genesis Key is present:

1. **Permission check** — Is actor allowed to perform this operation?
2. **Constraint check** — Are all constraints satisfied?
3. **Validation check** — Does this require review?

Operations failing these checks return:
- `E_PERMISSION_DENIED` — Actor lacks permission
- `E_CONSTRAINT_VIOLATED` — Constraint not satisfied

---

## Examples

### Personal Project

No Genesis Key needed. Full freedom.

### Small Team

```yaml
identity:
  name: "Team Project"
  owner: "human:lead"

actors:
  - id: "human:*"
    role: contributor
  - id: "agent:*"
    role: agent

permissions:
  contributor:
    - "*"
  agent:
    - capture
    - claim
    - release
    - submit
```

### Enterprise

```yaml
identity:
  name: "Corp Engineering"
  owner: "team:engineering-leads"

actors:
  - id: "human:lead-*"
    role: admin
  - id: "human:*"
    role: contributor
  - id: "agent:*"
    role: agent

permissions:
  admin:
    - "*"
  contributor:
    - capture
    - commit
    - claim
    - release
    - submit
  agent:
    - capture
    - claim
    - submit

constraints:
  close:
    require_evidence: true
  claim:
    require_claim: true

validation:
  tiers:
    tier_1:
      auto_close: true
    tier_2:
      auto_close: true
      review_window: 24h
    tier_3:
      auto_close: false
      require_human: true

  classification:
    - match: { tags: [security, compliance] }
      tier: tier_3
    - match: { tags: [production] }
      tier: tier_2
    - default: tier_1

federation:
  enabled: true
  sync_target: "https://corp.mentu.ai"
```

---

## Validation

Genesis Key is validated on:
- `mentu init` with `--genesis` flag
- `mentu genesis validate`
- First operation after Genesis Key modification

Invalid Genesis Key blocks all operations.

---

*Constitutional governance for commitment ledgers.*
