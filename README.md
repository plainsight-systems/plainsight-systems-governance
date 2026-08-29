# Plainsight Systems Governance

Engineering philosophies, C++ architecture and performance governance, and
repository workflow guidance used across Plainsight Systems LLC products.

This is the canonical, public source for these documents. Product repositories
link here rather than vendoring copies, so there is one version of each
document and no per-repo synchronisation.

## Contents

| Document | Purpose |
|---|---|
| [engineering_philosophies.md](./engineering_philosophies.md) | Engineering philosophies for product work. |
| [product_memory_workflow.md](./product_memory_workflow.md) | Project memory, queue, packets, review, and cross-session workflow. |
| [repo_creation_runbook.md](./repo_creation_runbook.md) | Runbook for bootstrapping or normalizing a product repo. |
| [cpp_architecture_playbook.md](./cpp_architecture_playbook.md) | C++ architecture and decomposition guidance. |
| [cpp_architecture_review.md](./cpp_architecture_review.md) | C++ architecture review checklist for humans and agents. |
| [cpp_performance_playbook.md](./cpp_performance_playbook.md) | C++ performance guidance for edge-native AI, real-time, and hot-path work. |
| [cpp_performance_review.md](./cpp_performance_review.md) | C++ performance review checklist for humans and agents. |

Related public material:

- [cpp-perf-guidelines](https://github.com/plainsight-systems/cpp-perf-guidelines) —
  technique-level C++ performance corpus. That corpus sits *below* the playbooks
  here: it owns concrete techniques, these documents own budgets, review gates,
  and acceptance criteria.
- [mcp-servers](https://github.com/plainsight-systems/mcp-servers) — MCP servers
  exposing that corpus to agents.

## What Is Not Here

Parent-organization policy — brand and entity boundaries, security-update and
end-of-life posture, and operational rules — is internal and lives in the
private operations repository. Brand-specific elaborations on the engineering
philosophies are internal for the same reason. Neither is required to read or
apply the documents in this repository.

## How Products Consume These Docs

Product repositories reference this repository by URL. They do not vendor
copies, and they do not commit symlinks into it.

A product repo should carry a single pointer — typically in `AGENTS.md` or
`docs/decisions/inherited.md` — naming this repository as the source of its
inherited governance. Documents can then be added, renamed, or revised here
without any downstream repository needing an edit.

Repos that keep a local checkout alongside this one may symlink these documents
into `docs/decisions/` for editor and agent convenience. Those links are a local
convenience and should be gitignored, not committed.

## Editing These Docs

Treat changes here as parent-organization decisions. Before editing, check:

- Which operating brands or product repos inherit the rule?
- Does the change conflict with product-specific decisions?
- Do product `MEMORY.md` indexes or `AGENTS.md` files need updates?

This is a documentation-only repository. Markdown only unless a future decision
explicitly adds tooling.

## License

The prose in this repository is licensed under
[Creative Commons Attribution 4.0 International](./LICENSE) (CC BY 4.0). Reuse
and adaptation are permitted, including commercially, with attribution.
