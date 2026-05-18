---
name: Plainsight repository creation runbook
description: Agent-consumable runbook for bootstrapping new Plainsight product repositories with Lite Factory governance.
type: project
---

# Repository Creation Runbook

Use this runbook when creating or normalizing a Plainsight product repository.
The goal is to give humans and agents the same starting surface every time:
clear project identity, inherited governance docs, local memory, active queue,
research capture, packet-based implementation, and review gates.

This runbook is operational. Follow it in order unless the target product has a
stricter local decision.

## Preconditions

Before editing a target repo, identify:

- product or operating-brand name,
- parent GitHub org and repository name,
- whether the repo is new or existing,
- primary implementation stack,
- whether the product is C++-dominant,
- whether the product has edge-native AI, real-time, or hot-path performance
  requirements,
- path to the local governance repo, normally
  `../plainsight-systems-governance` from sibling product repos.

If the repo is not new, inspect existing `AGENTS.md`, `CLAUDE.md`,
`docs/decisions/`, `docs/research/`, and README files before changing them.
Do not overwrite product-specific memory or decisions.

## Target Layout

Every active product repo should have this governance surface:

```text
AGENTS.md
CLAUDE.md
README.md
docs/
  decisions/
    MEMORY.md
    QUEUE.md
    workflow.md
    plainsight_policies.md -> ../../../plainsight-systems-governance/plainsight_policies.md
    engineering_philosophies.md -> ../../../plainsight-systems-governance/engineering_philosophies.md
    product_memory_workflow.md -> ../../../plainsight-systems-governance/product_memory_workflow.md
    repo_creation_runbook.md -> ../../../plainsight-systems-governance/repo_creation_runbook.md
    cpp_architecture_playbook.md -> ../../../plainsight-systems-governance/cpp_architecture_playbook.md
    cpp_architecture_review.md -> ../../../plainsight-systems-governance/cpp_architecture_review.md
    cpp_performance_playbook.md -> ../../../plainsight-systems-governance/cpp_performance_playbook.md
    cpp_performance_review.md -> ../../../plainsight-systems-governance/cpp_performance_review.md
    packets/
  research/
```

Non-C++ repos may omit the C++ symlinks only when they are not expected to
consume C++ guidance. C++-dominant repos should include all C++ symlinks.

## Bootstrap Commands

From the target repo root:

```sh
mkdir -p docs/decisions/packets docs/research

cd docs/decisions
ln -s ../../../plainsight-systems-governance/plainsight_policies.md .
ln -s ../../../plainsight-systems-governance/engineering_philosophies.md .
ln -s ../../../plainsight-systems-governance/product_memory_workflow.md .
ln -s ../../../plainsight-systems-governance/repo_creation_runbook.md .
ln -s ../../../plainsight-systems-governance/cpp_architecture_playbook.md .
ln -s ../../../plainsight-systems-governance/cpp_architecture_review.md .
ln -s ../../../plainsight-systems-governance/cpp_performance_playbook.md .
ln -s ../../../plainsight-systems-governance/cpp_performance_review.md .
cd ../..
```

If the governance repo is not a sibling of the product repo, adjust the symlink
target while preserving the same destination filenames.

For existing repos, create only missing directories and symlinks. Do not replace
a real file with a symlink until its contents have been reviewed and migrated.

## Root Files

Create or update `AGENTS.md` so agents know how to enter the repo.

Recommended minimum:

```markdown
# Agent Instructions

This repository inherits Plainsight Systems governance from
`docs/decisions/`.

## Session Entry

1. Read `docs/decisions/MEMORY.md`.
2. Read `docs/decisions/QUEUE.md`.
3. Read `docs/decisions/workflow.md`.
4. Read inherited governance docs relevant to the task.
5. Inspect product-specific source and tests before editing.

## Operating Rules

- Follow Lite Factory workflow from `docs/decisions/product_memory_workflow.md`.
- No non-trivial implementation without a packet in
  `docs/decisions/packets/`.
- Keep product decisions in `docs/decisions/` and research in
  `docs/research/`.
- Update `MEMORY.md` and `QUEUE.md` when work state or durable knowledge
  changes.
- Do not weaken inherited governance without an explicit decision.
```

Create or update `CLAUDE.md` as a tool-specific overlay. Keep it short and
point it back to the canonical docs.

Recommended minimum:

```markdown
# Claude/Codex Entry

Start with:

1. `docs/decisions/MEMORY.md`
2. `docs/decisions/QUEUE.md`
3. `docs/decisions/workflow.md`
4. Relevant inherited governance docs in `docs/decisions/`

Do not rely on chat history as project memory. Durable decisions belong in
`docs/decisions/`; investigations and lessons belong in `docs/research/`.
```

## Decision Memory Files

Create `docs/decisions/MEMORY.md`.

Recommended minimum:

```markdown
# Project Memory

This file is the canonical entry point for durable project context.

## Product Identity

- **Product:**
- **Operating brand:**
- **Parent entity:** Plainsight Systems LLC
- **Repository:**

## Inherited Governance

- `plainsight_policies.md`
- `engineering_philosophies.md`
- `product_memory_workflow.md`
- `repo_creation_runbook.md`

## C++ Governance

Include for C++-dominant products:

- `cpp_architecture_playbook.md`
- `cpp_architecture_review.md`
- `cpp_performance_playbook.md`
- `cpp_performance_review.md`

## Locked Decisions

- None yet.

## Research Index

- None yet.

## Active Workflow Pointers

- Queue: `QUEUE.md`
- Packets: `packets/`
- Workflow: `workflow.md`
```

Create `docs/decisions/QUEUE.md`.

Recommended minimum:

```markdown
# Work Queue

This file tracks active and accepted work.

## Active

- None.

## Ready

- None.

## Accepted

- None.

## Parking Lot

- None.
```

Create `docs/decisions/workflow.md`.

Recommended minimum:

````markdown
# Workflow

This repo uses Lite Factory workflow from
`product_memory_workflow.md`.

## Default Chain

```text
Coordinator
  -> packet
  -> implementation
  -> review
  -> QA
  -> MEMORY.md and QUEUE.md updates
```

## Local Notes

- Add product-specific workflow deviations here.
- Do not weaken inherited governance without a decision.
````

## Packet Creation

For the first real task in a repo, create a packet under
`docs/decisions/packets/`.

Packet filename convention:

```text
YYYY-MM-DD-short-kebab-title.md
```

Packet content should follow the shape in `product_memory_workflow.md`. For C++
work, include the C++ architecture note. For performance-sensitive C++ work,
include both the C++ architecture note and C++ performance note.

## Research Capture

Use `docs/research/` for:

- source investigations,
- implementation lessons,
- halted-work notes,
- performance measurements,
- architecture comparisons,
- vendor or dependency evaluations.

Research files should be dated when they are time-sensitive:

```text
YYYY-MM-DD-topic.md
```

Index durable research in `docs/decisions/MEMORY.md`.

## README Alignment

Every repo README should at minimum identify:

- product name,
- operating brand,
- product purpose,
- current maturity,
- setup or build entry point when known,
- governance entry point: `docs/decisions/MEMORY.md`.

Do not document aspirational behavior as shipped. Planned behavior should be
labeled planned, open, or future.

## Verification Checklist

Before accepting repo bootstrap, verify:

- `AGENTS.md` exists and points agents to memory, queue, and workflow.
- `CLAUDE.md` exists and points back to canonical project memory.
- `docs/decisions/MEMORY.md` exists.
- `docs/decisions/QUEUE.md` exists.
- `docs/decisions/workflow.md` exists.
- `docs/decisions/packets/` exists.
- `docs/research/` exists.
- inherited governance symlinks resolve.
- C++ governance symlinks exist for C++-dominant repos.
- `README.md` identifies the product and points to project memory.
- no product-specific decisions were overwritten.

Useful command:

```sh
test -f AGENTS.md
test -f CLAUDE.md
test -f docs/decisions/MEMORY.md
test -f docs/decisions/QUEUE.md
test -f docs/decisions/workflow.md
test -d docs/decisions/packets
test -d docs/research
find docs/decisions -maxdepth 1 -type l -exec sh -c '
  for link do
    [ -e "$link" ] || {
      echo "broken symlink: $link"
      exit 1
    }
  done
' sh {} +
```

## Commit And Remote

For a new repo:

```sh
git init -b main
git add README.md AGENTS.md CLAUDE.md docs
git commit -m "Bootstrap repo governance"
```

Create the remote in the correct GitHub org, normally `plainsight-systems`, and
push `main`.

For an existing repo, commit only the intended governance bootstrap files. Do
not mix repo-bootstrap work with product implementation.
