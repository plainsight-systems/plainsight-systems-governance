---
name: Plainsight Systems engineering philosophies
description: Engineering philosophies for Plainsight Systems product work.
type: project
---

# Engineering Philosophies

These philosophies apply across Plainsight Systems products. Product repos may
add local elaborations, but should not weaken these principles without an
explicit governance update.

## Real Product, Narrow Slice

Early products should demonstrate the real product on a narrow scope, not a
lower-fidelity demo of a future product.

This means:

- Narrow scope is acceptable.
- Lower quality inside the chosen scope is not.
- Contract relaxations must be defensible because the original contract was wrong, not because it was hard.
- "We can fix this after the demo" is a smell. Re-scope narrower or build the slice correctly.

## Hardware and Platform Specificity

When the product depends on a chosen platform, write to that platform honestly.
Do not add portability layers or abstractions for hypothetical future swaps.

This means:

- Pick blessed targets deliberately.
- Measure on the actual workload and actual target.
- Keep abstractions load-bearing.
- Treat popular defaults as leads, not decisions.

For Appario Native, this principle means the native installed app should use
platform-native capabilities where they materially improve merchant outcomes,
reliability, or trust. It should not remain a web app in a native wrapper unless
that is an explicit product decision.

## Merchant Utility Over SaaS Inertia

Appario started as a focused SaaS scan/report workflow. The native product is a
different shape: an installed merchant workbench with multiple functions.

This means:

- Product decisions should start from merchant workflows, not from preserving the old web platform.
- The installed app should earn its installation with persistent utility, local context, and operational convenience.
- The prior scanner/report slice is input material, not a constraint that defines the new product boundary.

## Test the Contract

Build-green is not proof. Tests and manual verification should pin the product
contract that matters.

This means:

- Non-trivial changes require deterministic verification.
- Tests must not mock away the risk that made the change meaningful.
- If a behavior cannot be tested, either redesign the behavior or document why the remaining risk is accepted.
