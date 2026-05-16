# Plainsight Systems Governance

Parent-organization policies, engineering philosophies, and repository workflow
guidance that apply across Plainsight Systems LLC operating brands and products.

## Contents

- [plainsight_policies.md](./plainsight_policies.md) - Parent-organization policies.
- [engineering_philosophies.md](./engineering_philosophies.md) - Engineering philosophies for product work.
- [product_memory_workflow.md](./product_memory_workflow.md) - Shared guidance for project memory, queue, packets, review, and cross-session workflow.
- [cpp_architecture_playbook.md](./cpp_architecture_playbook.md) - C++ architecture and decomposition guidance.
- [cpp_architecture_review.md](./cpp_architecture_review.md) - C++ architecture review checklist for humans and agents.
- [cpp_performance_playbook.md](./cpp_performance_playbook.md) - C++ performance guidance for edge-native AI, real-time, and hot-path work.
- [cpp_performance_review.md](./cpp_performance_review.md) - C++ performance review checklist for humans and agents.

## How Products Consume These Docs

Product repos should symlink the relevant docs into their own
`docs/decisions/` directory:

```sh
cd <product-repo>/docs/decisions
ln -s ../../../plainsight-systems-governance/plainsight_policies.md .
ln -s ../../../plainsight-systems-governance/engineering_philosophies.md .
ln -s ../../../plainsight-systems-governance/product_memory_workflow.md .
ln -s ../../../plainsight-systems-governance/cpp_architecture_playbook.md .
ln -s ../../../plainsight-systems-governance/cpp_architecture_review.md .
ln -s ../../../plainsight-systems-governance/cpp_performance_playbook.md .
ln -s ../../../plainsight-systems-governance/cpp_performance_review.md .
```

This keeps the source of truth here while making shared guidance visible in each
product's local decision memory.

## Editing These Docs

Treat changes here as parent-organization decisions. Before editing, check:

- Which operating brands or product repos inherit the rule?
- Does the change conflict with product-specific decisions?
- Do product `MEMORY.md` indexes or `AGENTS.md` files need updates?

This is a documentation-only repo. Markdown files only unless a future decision
explicitly adds tooling.
