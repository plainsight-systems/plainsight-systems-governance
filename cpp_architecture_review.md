---
name: Plainsight C++ architecture review
description: Agent and reviewer checklist for C++ architecture and decomposition review.
type: project
---

# C++ Architecture Review

Use this checklist for material C++ changes in Plainsight product repos. This
review is separate from syntax, formatting, clang-tidy, and ISO C++ best
practice checks.

The reviewer is evaluating whether the change looks like it was organized by a
senior C++ architect for this product architecture: C++ core functionality with
thin target-OS UI wrappers.

## Review Posture

Lead with architectural risks, not style preferences.

Prefer findings that identify:

- a bad dependency edge,
- a leaked implementation detail,
- unclear ownership or lifetime,
- misplaced product behavior,
- unstable public API,
- fake abstraction,
- missing test surface,
- platform-wrapper logic that belongs in core C++.

Do not require a split only because a file is long. Require a split only when
the current structure weakens cohesion, dependency control, API clarity,
testability, or platform separation.

## Required Inputs

Before reviewing, inspect:

- changed `.h`, `.hpp`, `.hh`, `.cpp`, `.cc`, `.cxx`, `.mm`, and platform UI
  wrapper files,
- related build files such as `CMakeLists.txt`, `.Build.cs`, `BUILD`,
  `BUILD.bazel`, package manifests, or project files,
- changed tests,
- dependency declarations,
- existing local architecture decisions in `docs/decisions/`.

If the repo has a dependency graph tool, include graph tool, or target graph
tool, use it for cross-component changes.

## Finding Format

Use this shape for findings:

```text
[P1] Core behavior moved into macOS wrapper
File: path/to/file.mm:123
Why this matters: The wrapper now owns product state transitions, so the core
C++ behavior cannot be tested or reused independently.
Expected fix: Move the state transition into the core component and keep the
wrapper responsible for UI event translation only.
```

Severity:

- `P0`: Blocks correctness, security, data safety, or product integrity.
- `P1`: Architectural regression likely to compound or block future work.
- `P2`: Maintainability, testability, dependency, or ownership risk.
- `P3`: Local cleanup or documentation issue.

## Boundary Checks

Ask:

- Did any target-OS UI wrapper gain domain logic, validation, persistence rules,
  protocol behavior, or state transitions?
- Did core C++ gain a dependency on UI framework types, application lifecycle
  objects, view models, controller classes, or OS presentation APIs?
- Are platform capabilities exposed through narrow adapters rather than broad
  framework leakage?
- Can the core behavior be tested without launching the target UI framework?

Flag:

- product behavior in SwiftUI/AppKit/UIKit/Win32/Android/Qt UI layers,
- core C++ includes of UI-wrapper headers,
- platform types in core public APIs without an explicit architecture decision,
- wrapper classes that own long-lived product state instead of reflecting core
  state.

## Component And File Checks

Ask:

- What is the component boundary?
- Is each new file attached to a stable concept?
- Is the principal class or function set cohesive?
- Are helper types private unless they are part of a stable public concept?
- Did the change introduce generic buckets such as `Manager`, `Service`,
  `Helper`, `Util`, `Common`, or `Misc` without a precise responsibility?

Flag:

- one-method abstractions that do not create a real boundary,
- public helper files used by only one implementation,
- unrelated responsibilities grouped only because they were implemented
  together,
- excessive splitting that makes the execution path harder to follow,
- hidden long-distance friendship or friend access across components.

## Dependency Checks

Ask:

- What dependency edges were added?
- Are new dependency edges in the correct direction?
- Are public dependencies only those required by public APIs?
- Are implementation-only dependencies private?
- Does any file rely on transitive includes or undeclared target dependencies?
- Did the change create or worsen a dependency cycle?

Flag:

- lower-level core depending on higher-level orchestration,
- core depending on UI wrapper code,
- build targets with unnecessary `PUBLIC` dependencies,
- public headers that include implementation-only headers,
- direct use of symbols from dependencies not declared in the local build
  target,
- cycles among libraries, modules, packages, or components.

## Header Checks

Ask:

- Are public headers self-contained?
- Does each implementation include its matching header first where practical?
- Do public headers expose only stable API?
- Could an expensive include become private without making the interface
  fragile?
- Are forward declarations used to clarify boundaries rather than hide real
  dependencies?

Flag:

- public headers that require prior includes,
- implementation details exposed in public headers,
- broad namespace imports in headers,
- macros that become part of public API accidentally,
- incidental standard-library or platform headers leaking through public API.

## Interface Checks

Ask:

- Are ownership, lifetime, threading, and error behavior clear from types or
  local contract documentation?
- Are raw pointers and references non-owning?
- Is ownership transfer represented explicitly?
- Are parameter groups strongly typed enough to prevent accidental misuse?
- Are preconditions and postconditions visible when they matter?

Flag:

- raw pointer ownership transfer,
- `shared_ptr` used where ownership should be singular,
- hidden global or singleton control inputs,
- adjacent same-type primitive parameters with different meanings,
- public APIs that require whole-program knowledge to use safely.

## Abstraction Checks

Ask:

- What real boundary does this abstraction represent?
- Could the same clarity be achieved with a private function, value type, or
  direct implementation?
- Is the abstraction stable enough to name?
- Is this interface needed for platform substitution, testing, ABI stability,
  hardware integration, plugin loading, or independent replacement?

Flag:

- abstract interfaces introduced only for neatness,
- dependency injection introduced without a real alternate implementation,
- class hierarchies used for ordinary code sharing,
- inheritance where composition would preserve simpler ownership and testing,
- interfaces whose lifetime or threading contract is unclear.

## Test Surface Checks

Ask:

- Does the test exercise core behavior without the UI wrapper?
- Does the test pin the product contract that made the change necessary?
- Are platform adapter tests separated from core behavior tests?
- Did the decomposition improve testability in a real way?

Flag:

- tests that only verify wrapper wiring while core behavior is untested,
- tests that mock away the risk introduced by the change,
- no deterministic test for new core behavior,
- test-only dependencies that create production dependency cycles.

## Acceptable Outcomes

A review may conclude:

- `approved`: no architecture findings.
- `approved_with_notes`: minor P3 issues only.
- `changes_requested`: any P0/P1, or P2 findings that should be fixed before
  merge.
- `needs_architecture_decision`: the change may be valid, but it conflicts with
  or extends product architecture and needs a decision record.

## Review Summary Template

```markdown
## C++ Architecture Review

Outcome: approved | approved_with_notes | changes_requested | needs_architecture_decision

### Findings

- [P?] ...

### Boundary Assessment

- Core/wrapper separation:
- Public/private dependency classification:
- Header/API surface:
- Ownership/lifetime clarity:
- Test surface:

### Residual Risk

- ...
```
