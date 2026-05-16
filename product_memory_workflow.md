---
name: Plainsight product memory and workflow
description: Shared project-memory and Lite Factory workflow guidance for Plainsight product repos.
type: project
---

# Product Memory and Workflow

Plainsight product repos should use a lightweight, repo-local memory system so
future sessions can reconstruct the product state without relying on chat
history.

## Required Project Memory Surface

Each active product repo should maintain:

| File | Purpose |
|---|---|
| `AGENTS.md` | Project-wide operator profile for humans and AI agents. |
| `CLAUDE.md` | Claude/Codex-specific overlay and session entry order. |
| `docs/decisions/MEMORY.md` | Index over locked decisions, research, and inherited governance docs. |
| `docs/decisions/QUEUE.md` | Current work queue and packet state. |
| `docs/decisions/workflow.md` | How implementation work moves through roles, review, QA, and acceptance. |
| `docs/decisions/packets/` | Scoped implementation packets. |
| `docs/research/` | Findings, investigations, lessons, and halted-work notes. |

`MEMORY.md` is the canonical entry point for context. `QUEUE.md` is the
canonical entry point for active work.

## Lite Factory Workflow

Until a product explicitly adopts the full Factory tooling, use Lite Factory:
artifact discipline without a tooling submodule.

Default chain:

```text
Coordinator
  -> scopes a packet in docs/decisions/packets/
  -> dispatches a Developer agent or implements directly for trivial docs
  -> obtains independent review for non-trivial implementation
  -> dispatches QA for non-trivial behavior
  -> updates QUEUE.md and MEMORY.md
```

## Non-Negotiables

- No implementation without a packet, except local decision-doc updates where the decision conversation is the work.
- One packet, one intent. No "while we're here" scope expansion.
- No facades. Unimplemented paths fail explicitly.
- Non-trivial changes require verification tied to the acceptance criteria.
- Non-trivial C++ changes require an architecture note before implementation
  and a C++ architecture review before acceptance.
- Reviewer and implementer should be distinct for material changes.
- QA identity should differ from the developer identity for non-trivial behavior.
- New decisions and lessons must be indexed in `MEMORY.md`.
- Active or accepted work must be reflected in `QUEUE.md`.

## Packet Shape

Use markdown packets:

```markdown
# <id>: <title>

**Status:** draft | ready | implementing | review_requested | changes_requested | review_approved | completed | accepted
**Change class:** trivial | local | cross-cutting | architectural

## Intent

- **What is changing:**
- **Why the change is necessary:**
- **Expected behavior changes:**
- **Guaranteed invariants/contracts:**

## Acceptance Criteria

- [ ] Deterministic verification item

## Verification Plan

## C++ Architecture Note

Required for non-trivial C++ changes. Omit only when the packet does not touch
C++ architecture.

- **Core components touched:**
- **Platform wrappers touched:**
- **Public interfaces changed:**
- **New files/classes/targets/modules:**
- **Dependency edges added or removed:**
- **Ownership/lifetime model:**
- **Test surface independent of UI wrappers:**
- **Why this decomposition is appropriate:**
- **What is intentionally not abstracted:**

## Records
```

The packet is the authority for the work. If the work changes, update or split
the packet before continuing.

## C++ Architecture Gate

For non-trivial C++ changes, use
`cpp_architecture_playbook.md` during packet shaping and
`cpp_architecture_review.md` during review.

The architecture review is distinct from code-style, compiler, linter, static
analysis, and QA checks. Its job is to evaluate component boundaries, core/UI
wrapper separation, dependency direction, public/private headers, ownership,
lifetime, abstraction quality, and testability.

Acceptance should be blocked when the review identifies a P0 or P1 architecture
finding. P2 findings may block acceptance when they affect shared components,
public API, dependency direction, or future testability.

## When to Escalate

Escalate instead of continuing when:

- a packet reveals a contradiction with a locked decision,
- a review finding challenges the premise of the packet,
- verification cannot honestly test the promised behavior,
- the implementation would require expanding scope,
- the product direction needs human judgment.
