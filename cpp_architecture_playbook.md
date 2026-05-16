---
name: Plainsight C++ architecture playbook
description: Architecture and decomposition guidance for Plainsight C++ product work.
type: project
---

# C++ Architecture Playbook

This playbook applies to Plainsight products whose dominant implementation is
C++ with thin target-OS UI wrappers. Product repos may add local rules, but
should not weaken these principles without an explicit governance update.

The purpose is to prevent agentic decomposition: code that satisfies local C++
best-practice checks while making weak architectural choices at the file,
component, module, or product-boundary level.

## Architectural Goal

Plainsight C++ products should have:

- cohesive core C++ components,
- explicit public interfaces,
- clear ownership and lifetime semantics,
- acyclic dependency direction,
- private implementation details,
- thin platform/UI wrappers,
- deterministic test surfaces independent of target UI frameworks.

File length is a secondary signal. A long cohesive implementation file can be
better than several short files that introduce fake abstractions, dependency
cycles, ambiguous ownership, or unstable public API.

## Source Basis

This playbook is grounded in public guidance from organizations and ecosystems
that invest heavily in C++:

- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines?lang=en)
- [Microsoft C++ Core Guidelines Checkers](https://learn.microsoft.com/en-us/cpp/code-quality/using-the-cpp-core-guidelines-checkers?view=msvc-170)
- [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
- [LLVM Coding Standards](https://llvm.org/docs/CodingStandards.html)
- [Unreal Engine Modules](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-modules?application_version=5.6)
- [Bloomberg BDE C++ Coding Standards](https://bloomberg.github.io/bde/knowledge_base/coding_standards.html)
- [AUTOSAR C++14 Guidelines](https://www.autosar.org/fileadmin/standards/R17-10_R1.2.0/AP/AUTOSAR_RS_CPP14Guidelines.pdf)
- [ROS 2 generated C++ interfaces](https://design.ros2.org/articles/generated_interfaces_cpp.html)
- [NASA C++ Coding Standards and Style Guide](https://ntrs.nasa.gov/citations/20080039927)

These sources differ on house style. The shared architectural signal is stable:
large C++ systems are governed through explicit interfaces, physical
components, dependency direction, header discipline, ownership clarity, and
private/public boundary control.

## Core And Wrapper Boundary

Product behavior belongs in platform-neutral C++ unless a product decision says
otherwise.

Concrete rules:

- Target-OS UI wrappers should orchestrate presentation, user input, platform
  lifecycle, permissions, notifications, and OS integration.
- Domain logic, state transitions, validation, persistence contracts, network
  protocols, deterministic transforms, and product rules belong in core C++.
- Core C++ must not depend on SwiftUI, AppKit, UIKit, Win32, WPF, Android UI,
  Electron, Qt UI widgets, or other target UI layers unless a product-specific
  architecture decision explicitly allows it.
- Platform capability should enter through narrow adapters whose contracts are
  owned by the core or an explicit platform-abstraction component.
- UI wrappers may depend inward on core C++; core C++ must not depend outward on
  UI wrappers.

## Physical Components

Treat a component as the smallest meaningful physical unit of design. A
component may be a conventional `.h`/`.cpp` pair, a CMake target, a library, a
package, or a product-defined module, but it must have a clear public contract.

Concrete rules:

- Every externally used component needs an identifiable public interface.
- Implementation-only helpers stay private to the component unless they are
  stable reusable concepts.
- A new file should be justified by cohesion, dependency control, testability,
  platform separation, or public/private boundary clarity.
- A new abstraction should name a real domain, platform, or technical boundary.
- Avoid generic "manager", "service", "helper", "util", "common", or
  "misc" names unless the responsibility is precise and hard to state better.
- Do not split a cohesive implementation solely because it crossed a line-count
  threshold.

## Dependency Direction

Dependency direction is an architectural contract, not a build-system detail.

Concrete rules:

- Component, package, target, and module dependency graphs should be acyclic.
- Lower-level core components must not depend on higher-level orchestration,
  UI, or application-entry components.
- Public dependencies must be dependencies that appear in or are required by
  the public interface.
- Implementation-only dependencies should be private.
- Do not rely on transitive dependencies. If a file directly uses an entity, it
  must include or depend on the component that provides that entity according to
  the local build system.
- A dependency edge that simplifies one implementation while making the graph
  harder to reason about is usually a bad trade.

## Header Discipline

Headers are architecture. A public header controls compile-time cost, consumer
coupling, ABI/API surface, and future change freedom.

Concrete rules:

- Public headers must be self-contained.
- Implementation files should include their corresponding header first when the
  codebase structure supports that pattern.
- Headers should expose the smallest stable API needed by consumers.
- Public headers must not expose implementation-only dependencies.
- Expensive or unstable includes should be moved out of public headers when a
  cleaner boundary is available.
- Forward declarations are allowed when they reduce public coupling without
  hiding a real dependency or making the interface fragile.
- Avoid namespace imports, broad macros, and incidental helper declarations in
  public headers.

## Interface Design

Interfaces should make assumptions visible.

Concrete rules:

- Prefer explicit, strongly typed interfaces over weak primitive clusters.
- State ownership transfer through type choice, not comments alone.
- Do not transfer ownership through raw pointers or references.
- Use raw pointers and references for non-owning access only when the lifetime
  contract is clear at the call site.
- Avoid non-const globals, hidden singletons, and ambient state as control
  inputs.
- Keep public function argument lists short enough to remain readable and hard
  to misuse. Introduce a named parameter object when it represents a stable
  concept, not merely to hide an overgrown signature.
- Public APIs should document preconditions, postconditions, error behavior,
  threading expectations, and lifetime expectations when those are not obvious
  from the type system.

## Ownership And Lifetime

Ownership is part of architecture because it determines which component is
responsible for cleanup, mutation, synchronization, and long-term object
validity.

Concrete rules:

- Prefer value semantics and RAII where practical.
- Prefer `std::unique_ptr` or equivalent project-local ownership types for
  exclusive dynamic ownership.
- Use shared ownership only when the shared lifetime is real and documented.
- `std::shared_ptr` must not be used to avoid deciding which component owns an
  object.
- Observer relationships should be represented distinctly from owning
  relationships.
- Cross-thread ownership and callback lifetime must be explicit enough to
  review without whole-program archaeology.

## Abstract Interfaces And Inheritance

Abstract interfaces are useful at real boundaries. They are harmful when used as
ornament.

Concrete rules:

- Introduce an abstract interface only for a real architectural boundary:
  platform adapter, hardware adapter, plugin boundary, test seam, ABI boundary,
  independently replaceable implementation, or cross-module contract.
- Prefer composition over inheritance for ordinary implementation reuse.
- Multiple inheritance should be limited to interface-style bases or to a
  product-specific pattern with clear justification.
- Inheritance hierarchy design must distinguish interface inheritance from
  implementation inheritance.
- A virtual interface should have a clear ownership, lifetime, and threading
  contract.

## Decomposition Rules

Decompose when the split improves the architecture.

Good reasons to split:

- A stable concept has emerged and has a name.
- A dependency can become private.
- A public API can become smaller or more coherent.
- A platform-specific concern can be isolated.
- A deterministic test surface becomes possible.
- A dependency cycle is removed.
- Multiple callers need the same cohesive behavior.

Bad reasons to split:

- The file is long but still cohesive.
- The agent wants symmetrical-looking files.
- A helper could be reused hypothetically.
- The split creates a one-method class with no independent responsibility.
- The split moves private implementation details into a public header.
- The split makes ownership or lifetime less obvious.

## Review Expectations

For non-trivial C++ changes, the implementation packet or review record should
answer:

- Which core components changed?
- Which platform wrappers changed?
- What public interfaces changed?
- What new files, classes, targets, or modules were added?
- Why is the chosen decomposition better than modifying existing components?
- What dependency edges were added or removed?
- Which dependencies are public versus private?
- What owns new objects and resources?
- What test surface proves the core behavior independently of UI wrappers?
- What was intentionally not abstracted?

## Automated Fitness Checks

Product repos should automate the checks that can be automated:

- public headers compile standalone,
- implementation files include their matching header first,
- include-what-you-use or equivalent dependency checks,
- CMake/Bazel target dependency correctness,
- public/private dependency classification,
- forbidden dependency edges from core to UI wrappers,
- dependency-cycle detection,
- compile-time impact tracking for large changes,
- ABI/API surface monitoring for exported libraries,
- static analysis for ownership, lifetime, and C++ Core Guidelines profiles.

Automation does not replace architecture review. It narrows the review to the
judgment calls humans and agents are most likely to get wrong.
