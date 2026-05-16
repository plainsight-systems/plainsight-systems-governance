---
name: Plainsight C++ performance review
description: Agent and reviewer checklist for C++ performance-sensitive changes.
type: project
---

# C++ Performance Review

Use this checklist for material C++ changes that touch hot paths, edge-native
AI, real-time behavior, high-volume data processing, startup/model-load time, or
memory-sensitive code.

This review is separate from architecture review and C++ Core Guidelines checks.
The reviewer is asking whether the implementation preserves the reason the
product chose C++.

## Review Posture

Lead with measured risk.

Prefer findings that identify:

- missing target-hardware measurement,
- regression against a stated budget,
- avoidable allocation or copy,
- cache-hostile data layout,
- unnecessary dispatch or abstraction in a hot path,
- synchronization or I/O in a latency-sensitive path,
- AI runtime configuration that ignores latency/throughput goals,
- benchmark that omits preprocessing, postprocessing, or data movement.

Do not ask for clever low-level code without evidence. Simple code that meets
the budget is better than clever code without a measured reason.

## Required Inputs

Before reviewing, inspect:

- changed C++ and platform-wrapper files,
- build configuration and compiler flags,
- benchmark or profiling notes,
- target hardware notes,
- changed inference/model/runtime configuration,
- changed tests and performance checks,
- relevant packet performance note.

For edge-native AI, inspect preprocessing, tensor creation, runtime invocation,
postprocessing, and result transfer to the UI or caller.

## Finding Format

Use this shape for findings:

```text
[P1] Hot inference loop allocates per frame
File: path/to/pipeline.cpp:123
Why this matters: This path runs once per camera frame on the target device.
The new vector growth and string conversion add allocator jitter and memory
traffic outside the stated 16 ms frame budget.
Expected fix: Preallocate the tensor/result buffers during pipeline setup and
reuse them across frames; keep label conversion outside the steady-state path.
```

Severity:

- `P0`: Breaks a hard real-time, safety, data-integrity, or product-critical
  performance requirement.
- `P1`: Likely performance regression in a hot path or missing measurement for
  a performance-sensitive change.
- `P2`: Maintainability/performance risk that can compound or hide future
  regressions.
- `P3`: Local cleanup, measurement clarity, or documentation issue.

## Budget And Measurement Checks

Ask:

- What exact budget is being protected?
- What target hardware and workload were used?
- Is there a baseline?
- Are before/after numbers provided?
- Were release-like compiler flags used?
- Does the benchmark include warm-up behavior when steady state is claimed?
- Does the benchmark include all work visible to the user or product contract?

Flag:

- performance-sensitive change with no measurement,
- benchmark only on developer workstation when product target is edge hardware,
- model-only benchmark for an AI pipeline whose preprocessing dominates,
- average-only results when tail latency or jitter matters,
- debug or instrumentation-heavy build used as proof.

## Hot Path Checks

Ask:

- Which functions are hot?
- How often do they run?
- What data volume do they process?
- Did the change add abstraction, allocation, logging, dispatch, or conversion
  to the hot path?
- Can setup work move out of the steady-state path?

Flag:

- runtime configuration lookup per item,
- logging or metrics allocation per item,
- repeated string formatting or parsing,
- repeated container growth,
- repeated virtual/type-erased calls inside large loops,
- lazy initialization on first hot-path use.

## Data Layout Checks

Ask:

- Is data stored in the order it is processed?
- Are hot fields packed separately from cold metadata?
- Does the loop walk contiguous memory?
- Does the layout support SIMD, batching, or vectorized runtime kernels?
- Are pointer chains or object graphs causing scattered access?

Flag:

- per-element heap objects for bulk processing,
- maps or string-keyed lookups in inner loops,
- array-of-objects layout when only one field is read across many objects,
- cold metadata carried through every hot operation,
- copies introduced only to satisfy a generic API.

## Allocation Checks

Ask:

- Are there allocations in real-time, per-frame, per-token, per-sample, or
  high-frequency callback paths?
- Are buffers sized and reused?
- Are allocation counts measured?
- Are allocator choices explicit for sustained workloads?
- Does the change increase peak or steady-state memory?

Flag:

- `new`, `make_shared`, `std::function`, growing `std::vector`, growing
  `std::string`, or map insertion in hot paths,
- `shared_ptr` ownership graphs in performance-sensitive object lifetimes,
- unbounded queues,
- per-callback scratch allocation,
- runtime shape changes that force reallocation.

## Copy And Boundary Checks

Ask:

- How many times does a payload move or copy?
- Are pipeline boundaries passing ownership, views, or copies?
- Are serialization/deserialization steps necessary?
- Does the UI wrapper receive only the data it needs?
- Are tensor/image/audio buffers kept in runtime-native layout?

Flag:

- large payload copied through wrapper/view-model layers,
- JSON/protobuf/string conversion in steady-state processing,
- tensor copied CPU->GPU->CPU unnecessarily,
- image/audio buffers reformatted multiple times,
- zero-copy claim without pointer/lifetime evidence.

## Concurrency Checks

Ask:

- Is concurrency protecting latency, throughput, or responsiveness?
- Are locks and atomics outside the hot path where possible?
- Is work bounded?
- Are producer/consumer queues backpressured?
- Does parallelism improve the measured budget?
- Is latency-sensitive work isolated from background work?

Flag:

- unbounded thread creation,
- blocking I/O in hot paths,
- lock contention without measurement,
- atomics in inner loops without a reason,
- background work stealing CPU/GPU/NPU budget from foreground inference,
- queues that can grow without bound under load.

## Edge AI Checks

Ask:

- Which runtime/backend/execution provider is used on the target device?
- Is the runtime configured for latency or throughput intentionally?
- Are graph optimizations enabled?
- Is model compilation or context creation outside the hot path?
- Are tensor shapes stable?
- Are preprocessing and postprocessing measured?
- Is batching justified by the product goal?
- Are precision/quantization choices validated for accuracy?

Flag:

- default runtime configuration accepted without measurement,
- synchronous inference where the runtime recommends async for production
  pipelines and overlap is possible,
- batch size selected for benchmark optics rather than product behavior,
- changing input shapes on every request,
- rebuilding sessions, engines, compiled models, execution contexts, or tensor
  buffers repeatedly,
- accuracy not checked after quantization or lower precision.

## Enterprise Bloat Checks

Ask:

- Could this hot-path abstraction be a direct function or data transform?
- Is this generic mechanism needed in production, or only in setup/tooling?
- Does the design optimize for framework neatness over hardware behavior?

Flag:

- service/container lookup in hot paths,
- abstract factory or plugin mechanism where the implementation is fixed,
- generic message bus for local calls,
- reflection or dynamic property access in inner loops,
- exception-based expected control flow,
- object-oriented decomposition that prevents contiguous data processing.

## Acceptable Outcomes

A review may conclude:

- `approved`: no performance findings.
- `approved_with_notes`: minor P3 issues only.
- `changes_requested`: any P0/P1, or P2 findings that should be fixed before
  merge.
- `needs_performance_decision`: the change may be valid, but it changes a
  performance budget, target hardware assumption, or product-level latency /
  throughput tradeoff.

## Review Summary Template

```markdown
## C++ Performance Review

Outcome: approved | approved_with_notes | changes_requested | needs_performance_decision

### Findings

- [P?] ...

### Performance Assessment

- Target hardware/workload:
- Budget:
- Baseline:
- Before/after result:
- Hot paths changed:
- Allocation/copy behavior:
- Concurrency behavior:
- Edge AI runtime configuration:

### Residual Risk

- ...
```
