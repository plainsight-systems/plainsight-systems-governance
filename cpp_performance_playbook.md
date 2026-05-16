---
name: Plainsight C++ performance playbook
description: Performance guidance for Plainsight C++ product work, especially edge-native AI and real-time workloads.
type: project
---

# C++ Performance Playbook

This playbook applies to Plainsight products that choose C++ because the
product should not leave meaningful compute on the table. That posture is the
Doom Philosophy: target the real machine, understand the hot path, and spend
abstraction only where it pays for itself.

This document complements `cpp_architecture_playbook.md`. Architecture review
asks whether the system is decomposed correctly. Performance review asks
whether that decomposition preserves locality, predictability, and hardware
utilization.

## Source Basis

This playbook is grounded in public performance material from game-engine,
robotics, and edge-AI ecosystems:

- [Unreal Engine performance profiling](https://dev.epicgames.com/documentation/en-us/unreal-engine/introduction-to-performance-profiling-and-configuration-in-unreal-engine?application_version=5.6)
- [Unreal Insights](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-insights-in-unreal-engine?application_version=5.6)
- [ROS 2 real-time background](https://design.ros2.org/articles/realtime_background.html)
- [ROS 2 real-time proposal](https://design.ros2.org/articles/realtime_proposal.html)
- [ROS 2 efficient intra-process communication](https://docs.ros.org/en/rolling/Tutorials/Demos/Intra-Process-Communication.html)
- [ROS 2 intra-process communication design](https://design.ros2.org/articles/intraprocess_communications.html)
- [NVIDIA TensorRT performance best practices](https://docs.nvidia.com/deeplearning/tensorrt/10.16.0/performance/best-practices.html)
- [NVIDIA TensorRT performance optimization](https://docs.nvidia.com/deeplearning/tensorrt/latest/performance/optimization.html)
- [ONNX Runtime performance tuning](https://onnxruntime.ai/docs/performance/tune-performance/)
- [ONNX Runtime graph optimizations](https://onnxruntime.ai/docs/performance/model-optimizations/graph-optimizations.html)
- [OpenVINO inference optimization](https://docs.openvino.ai/2024/openvino-workflow/running-inference/optimize-inference.html)
- [OpenVINO high-level performance hints](https://docs.openvino.ai/2024/openvino-workflow/running-inference/optimize-inference/high-level-performance-hints.html)
- [Mike Acton, Data-Oriented Design and C++](https://isocpp.org/blog/2015/01/cppcon-2014-data-oriented-design-and-c-mike-acton)
- [Naughty Dog, Parallelizing the Naughty Dog Engine Using Fibers](https://gdcvault.com/play/1022186/Parallelizing-the-Naughty-)
- [Insomniac core coding standard archive](https://gist.github.com/Kerollmops/fcad27cfef9e3552cb75a3d201494ba6)
- [id Software DOOM source](https://github.com/id-Software/DOOM)
- [id Software DOOM 3 source](https://github.com/id-Software/DOOM-3)
- [Guerrilla, Streaming the World of Horizon Zero Dawn](https://www.guerrilla-games.com/read/Streaming-the-World-of-Horizon-Zero-Dawn)
- [Capcom RE ENGINE overview](https://www.capcom.co.jp/ir/english/data/oar/2025/re-engine.html)

Some current proprietary engines named in product discussions, including id
Tech 8, Naughty Dog's current internal engine, and Capcom's current RE ENGINE,
do not publish enough low-level C++ rules to treat them as direct style guides.
Use public talks, source drops, and official documentation as evidence, and
mark any inference as an inference.

## Performance Goal

Plainsight performance-sensitive C++ should:

- define latency, throughput, memory, and power budgets before optimizing,
- measure on target hardware and target data,
- minimize allocation, copies, serialization, and dispatch on hot paths,
- preserve cache locality and predictable memory access,
- keep hot data compact and cold data out of tight loops,
- use concurrency deliberately, with bounded synchronization,
- choose inference/runtime configuration from measured workload goals,
- reject enterprise C++ bloat in hot paths.

## Measurement First

Performance work without measurement is guesswork.

Concrete rules:

- Every performance-sensitive packet must state the target hardware and workload.
- Define which metric matters: latency, throughput, jitter, memory footprint,
  power, startup time, model-load time, or sustained thermals.
- Capture a baseline before changing the design.
- Measure release-like builds with relevant compiler flags.
- Warm up runtimes when measuring steady-state performance.
- Record enough environment context to explain results: device, OS, CPU/GPU/NPU,
  clocks or power mode where available, model, input shape, batch size, and data
  size.
- Treat profiler overhead as real. Use it to locate work, then confirm with
  minimally intrusive timing or benchmark runs.
- Do not accept a performance change because it "should be faster." Accept it
  because it improves the named budget without breaking the product contract.

## Hot Path Ownership

Hot paths need named ownership.

Concrete rules:

- Identify hot paths in packet notes and code review.
- Keep hot-path code readable, but do not force it through generic enterprise
  abstractions for symmetry.
- Document why a hot-path abstraction exists when it introduces virtual
  dispatch, type erasure, heap allocation, locks, atomics, string processing,
  JSON/protobuf parsing, logging, or dynamic lookup.
- Keep configuration, validation, logging, reflection, and diagnostics out of
  steady-state loops unless explicitly budgeted.
- Prefer direct data transforms over command buses, observer chains, generic
  registries, and layered service calls in hot code.

## Data-Oriented Design

For performance-sensitive code, design around the data transformation.

Concrete rules:

- Start with the input data, output data, access pattern, and operation count.
- Prefer contiguous storage for data iterated together.
- Split hot data from cold metadata when the hot loop does not need the full
  object.
- Prefer structure-of-arrays or packed arrays when repeated operations touch the
  same field across many elements.
- Avoid pointer-rich object graphs in hot paths.
- Avoid per-element polymorphic dispatch in large loops unless measured and
  justified.
- Choose data layouts that make SIMD, batching, prefetching, and cache reuse
  possible.
- Do not hide an expensive data layout behind a clean object API and call it
  done.

## Allocation Discipline

Allocation strategy is a design decision.

Concrete rules:

- Avoid dynamic allocation in real-time, per-frame, per-sample, per-token,
  per-detection, and high-frequency callback paths.
- Preallocate buffers for known maximums or bounded working sets.
- Reuse scratch buffers when lifetime and threading make that safe.
- Prefer arenas, pools, ring buffers, fixed-capacity containers, or
  runtime-specific allocators when allocation frequency is part of the workload.
- Track peak memory, allocation count, and allocation size distribution for
  performance-sensitive changes.
- Do not use `std::shared_ptr`, `std::function`, `std::string`,
  `std::vector` growth, or associative containers casually in hot paths.
- When dynamic allocation remains necessary, make the reason and budget visible.

## Copies, Serialization, And Boundaries

Copies are often the hidden tax in C++ systems that look clean on paper.

Concrete rules:

- Count data movement across pipeline stages, thread boundaries, process
  boundaries, model-runtime boundaries, and UI-wrapper boundaries.
- Prefer move-only ownership transfer for large payloads when a single consumer
  owns the next stage.
- Use zero-copy or shared-memory mechanisms when the payload size and frequency
  justify the added complexity.
- Avoid converting between JSON, protobuf, strings, tensors, images, and custom
  structs inside steady-state hot paths.
- Keep data in the runtime-native layout once it enters an inference or signal
  pipeline.
- Do not bounce large payloads through UI-layer data models.

## Concurrency And Real-Time Predictability

Concurrency should reduce elapsed work without increasing jitter beyond the
budget.

Concrete rules:

- Prefer bounded job systems, task queues, or explicit pipelines over unbounded
  thread creation.
- Avoid locks in real-time or high-frequency hot paths unless contention is
  measured and bounded.
- Avoid blocking I/O in hot paths.
- Avoid page faults, lazy initialization, first-use allocation, and runtime
  plugin loading in real-time loops.
- Use atomics deliberately; they are synchronization and can be expensive.
- Separate latency-sensitive work from throughput-oriented background work.
- Document thread ownership for mutable state.
- Prove that parallelism improves the named budget; do not add concurrency only
  because work can be split.

## Edge-Native AI

Edge AI performance is usually a pipeline problem, not only a model problem.

Concrete rules:

- Decide whether the product optimizes for single-request latency, sustained
  throughput, memory footprint, power, or thermal stability.
- Select the execution provider or runtime backend explicitly for the target
  device.
- Use runtime graph optimizations and offline model formats where available.
- Use quantization, lower precision, tensor-core/NPU-friendly shapes, or
  runtime-specific compilation only when accuracy and product behavior remain
  acceptable.
- Avoid changing input shapes or optimization profiles in the steady state when
  the runtime pays setup costs for shape/resource recomputation.
- Batch only when it improves the product's actual budget. Batching can improve
  throughput while hurting latency.
- Prefer asynchronous inference APIs for production pipelines when the runtime
  recommends them and the product can use overlap safely.
- Keep preprocessing, postprocessing, tensor marshaling, and memory transfers in
  the benchmark, not just model execution.
- Track model load time, warm-up behavior, peak memory, steady-state memory,
  and device utilization.
- Cache compiled models, execution contexts, tensor buffers, and reusable
  runtime resources where supported.

## Enterprise C++ Bloat Signals

Performance-sensitive C++ is drifting when hot paths accumulate:

- service locators, dependency containers, or abstract factories,
- deep interface stacks over concrete data transforms,
- dynamic casts or RTTI in high-frequency paths,
- `std::function` or type-erased callbacks per item,
- shared ownership graphs,
- logging, tracing, metrics, or allocation on every item without sampling or
  compile-time control,
- string keys, maps, JSON, or protobuf in steady-state inner loops,
- exception-based control flow in expected hot-path cases,
- tiny polymorphic objects spread across the heap,
- general-purpose message buses for local in-process calls,
- configurable code paths where the production path is known.

These patterns may be acceptable in orchestration, tooling, setup, tests, and
low-frequency control planes. They require justification in hot paths.

## Performance Acceptance

A performance-sensitive C++ change should include:

- target hardware and workload,
- baseline measurement,
- performance budget,
- changed hot paths,
- allocation/copy/serialization analysis,
- concurrency model,
- inference runtime configuration when AI is involved,
- before/after results,
- residual risk.

For edge-native AI, include at least one end-to-end measurement that covers
preprocessing, runtime execution, postprocessing, and data movement.

## Automated Fitness Checks

Product repos should add lightweight performance fitness checks where possible:

- microbenchmarks for stable kernels,
- end-to-end benchmarks for AI pipelines,
- allocation-count regression checks,
- model-load and warm-up timing checks,
- maximum payload-copy checks in large-data paths,
- dependency checks that prevent UI wrappers from owning hot data transforms,
- perf counters or profiler traces for known bottleneck classes,
- CI warnings when benchmark deltas exceed a configured threshold.

Performance CI should detect regressions, not pretend to prove universal
performance. Human review still owns the interpretation.
