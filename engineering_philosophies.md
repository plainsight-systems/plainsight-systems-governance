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

### The Apple II standard

The reference point is the Apple II. It shipped as the real product at full
intended quality. It had limitations — memory, peripherals, software ecosystem
— but every part that shipped was the product as conceived, not a placeholder
for a better version later. Markkula funded the product as it stood, not a demo
of a future product. The Macintosh launch was the Macintosh, not a Macintosh
demo. The investor, and the first customers, saw what the product *was* — not
what it would later become.

An early-stage demonstration is therefore a demonstration *of the real
product*, at full intended quality, on a deliberately narrow slice. The slice
is narrow on purpose; the quality on that slice is uncompromising. "Vision over
feasibility" is about choosing the right product to build and the right slice
to show first — never about lowering the fidelity of what ships in that slice.

### Demo-ware is forbidden

"Demo-ware" — a lower-fidelity preview of the real product, with "good enough
for now" workarounds slated to be fixed for the commercial release — is the
opposite of this philosophy and is forbidden.

- "Good enough for the demo" and "we can fix this for the launch" are smells,
  not plans. The framing itself is the warning sign.
- A contract or acceptance criterion may be relaxed only when the relaxation is
  *honest*: the original contract was a category error and the correct contract
  is genuinely different. "This is too hard to finish in time" is not a valid
  relaxation rationale. "This contract was wrong; the correct contract is X" is.
- **A contract relaxation must be defensible in language that does not contain
  the word "demo."** If the rationale falls apart without "for the demo we
  can…", it is demo-ware framing in disguise — reject it, and either re-scope
  narrower or invest the work to honor the original contract.
- When the early stage ends there is no upgrade step from "demo quality" to
  "real quality." There was never a demo quality — only the real product,
  demonstrated on a narrow scope, expanding to broader scope over time. If a
  plan contains such an upgrade step, the early deliverable was demo-ware.

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
