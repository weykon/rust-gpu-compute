---
name: guide
description: Rust GPU Compute Development Guide — helps developers write GPU-accelerated Rust code with correct data-parallel patterns and awareness of GPU execution constraints. Use when the user mentions GPU compute, WGSL, compute shaders, wgpu, boids, particle systems, simulations, data-parallel Rust, or VectorWare-style Rust runtimes.
---

# Rust GPU Compute Development Guide

## Background

This skill is based on a simple but important idea:

> A Rust-flavored API can make GPU programming easier to write, but it does not remove the GPU execution model.

If a runtime lets you express GPU work with Rust code, you still need to respect warp behavior, memory bandwidth, synchronization costs, and host↔device transfer overhead.

This guide is qualitative rather than benchmark-driven. It is intended to help with reasoning and review, not to provide universal thresholds or hardware-independent performance guarantees.

## Scope and non-goals

This guide is mainly for qualitative reasoning and code review.

It is intended to help answer questions like:
- is this workload data-parallel enough to be a GPU candidate?
- are transfer boundaries, memory layout, and control flow likely to dominate the result?
- is a Rust-on-GPU runtime improving ergonomics without hiding the underlying execution model?

It is not intended to claim:
- a universal replacement for WGSL or shader programming in general
- hardware-independent break-even thresholds
- that all Rust concurrency abstractions map cleanly or efficiently to GPU execution

## Core constraints

### 1. Branch divergence

Threads inside the same warp execute together. If some threads take one branch and others take another branch, the warp often serializes those paths.

**Bad pattern:**

```wgsl
if (local_id == 0u) {
    sum += values[i];
} else {
    sum -= values[i];
}
```

**Better pattern:**

```wgsl
sum += values[i];
```

**Practical guidance:**
- split highly different work into separate kernels
- prefer uniform work per thread when possible
- convert control flow into data flow when it improves warp coherence

### 2. Memory transfer overhead

GPU acceleration is not free. Uploading data to the GPU and downloading results has a fixed cost.

**Rule of thumb:** if the dataset is small or the computation is short-lived, CPU execution is often faster.

**Bad pattern:**

```rust
for batch in data.chunks(100) {
    gpu.upload(batch);
    gpu.run();
    gpu.download();
}
```

**Better pattern:**

```rust
let device_buffer = gpu.upload(&data);
for _ in 0..iterations {
    gpu.run(&device_buffer);
}
let result = gpu.download(&device_buffer);
```

**Practical guidance:**
- batch transfers instead of sending tiny chunks repeatedly
- keep data resident on the GPU across multiple passes
- only move results back when the CPU actually needs them

### 3. Rust feature limits on GPU

Depending on the stack, backend, and runtime, GPU-side Rust often supports a restricted subset of the language or makes some language features impractical in hot device code.

Common limitations or pressure points include:
- heap allocation is often unavailable, discouraged, or heavily constrained on device
- recursion is often unsupported or undesirable
- `unwrap()` / `panic!()` behavior is often unsupported, undesirable, or highly runtime-specific on device
- dynamic-size abstractions may be limited or compile down poorly depending on the toolchain
- `std` support is often reduced or replaced by a more restricted environment

**Preferred mindset:**
- preallocate buffers
- use fixed-size or explicit layouts
- handle fallible logic on the host side
- keep kernels simple and predictable

### 4. Memory coalescing

Adjacent threads should access adjacent memory addresses. If thread access is strided or random, memory bandwidth drops quickly.

**Bad pattern:**

```rust
let value = data[global_id * STRIDE];
```

**Better pattern:**

```rust
let value = data[global_id];
```

**Practical guidance:**
- prefer structure-of-arrays layouts when it improves contiguous access
- avoid scattered reads unless there is a strong reason
- design buffers for the GPU, not just for CPU ergonomics

### 5. CPU vs GPU fit

Not every compute task should go to the GPU.

| Workload | Best default |
| --- | --- |
| large map / reduce style transforms | often GPU |
| image processing / filters | often GPU |
| Monte Carlo simulation | often GPU |
| particle updates | often GPU |
| heavy branchy business logic | often CPU |
| small, short-lived workloads with low arithmetic intensity | often CPU |
| irregular pointer-chasing | often CPU |
| nested, data-dependent control flow | usually CPU |

## VectorWare-style runtime vs WGSL vs a transpiler

These categories are not interchangeable. They refer to different layers of the stack: shader authoring, code generation / compilation, and runtime execution model.

```text
Traditional WGSL path
Rust host code -> WGSL kernel -> GPU

VectorWare-style runtime path
Rust host code -> Rust thread-like API -> runtime lowering -> GPU warp execution

Rust-to-WGSL transpiler goal
Rust compute DSL -> generated WGSL / SPIR-V -> GPU

The point of this comparison is to separate layers, not to imply that all three approaches are equally mature, equally portable, or interchangeable in practice.
```

### What changes with a Rust runtime on top of the GPU?

A VectorWare-style model should be read as one runtime approach, not as a universal replacement for shader programming. It can make some Rust concurrency patterns feel more natural, but it does not erase GPU-specific resource limits or performance trade-offs.

You get:
- a more familiar language surface
- easier integration with Rust codebases
- less context switching between host code and shader code

You do **not** get:
- freedom from warp divergence
- freedom from transfer cost
- freedom from GPU memory layout concerns
- full general-purpose CPU semantics on device

## Boids and particle systems

Boids are a good example of code that looks easy on the CPU but needs restructuring for the GPU.

**CPU-shaped version:**

```rust
fn update_boid(boid: &mut Boid, neighbors: &[Boid]) {
    for neighbor in neighbors {
        if distance(boid, neighbor) < RANGE {
            boid.velocity += separation(boid, neighbor);
        }
    }
}
```

This is usually not GPU-friendly because it combines:
- nested neighbor traversal
- distance-based branching
- irregular memory access

**More GPU-friendly direction:**
- build a spatial hash or uniform grid first
- restrict neighbor checks to nearby cells
- split the simulation into phases
- use parallel reduction for shared aggregates
- keep each kernel focused on one uniform step

In practice, a scalable boids pipeline often looks more like: build grid -> index or sort -> neighbor pass -> integrate. The exact shape depends on the simulation size and memory budget, but naive all-pairs traversal stops scaling quickly.

## Recommended review flow

When helping with Rust GPU compute code:

1. Identify the unit of parallel work.
2. Check whether threads do roughly uniform work.
3. Inspect transfer boundaries between CPU and GPU.
4. Check memory layout and access pattern.
5. Remove or isolate divergent branches.
6. Decide whether the workload should stay on CPU.

## Development checklist

Before moving a Rust workload to the GPU, check:

- [ ] Is the dataset large enough to amortize transfer cost?
- [ ] Is the workload data-parallel?
- [ ] Can the hot path avoid deep branching?
- [ ] Can buffers be preallocated?
- [ ] Is memory access contiguous or close to contiguous?
- [ ] Can the work be split into clear GPU phases?
- [ ] Is GPU execution actually better than a SIMD / multithreaded CPU version?

## References

- [VectorWare: Threads on GPU](https://www.vectorware.com/blog/threads-on-gpu/)
- [wgpu](https://github.com/gfx-rs/wgpu)
- [rust-gpu](https://github.com/EmbarkStudios/rust-gpu)
- [WGSL specification](https://www.w3.org/TR/WGSL/)

## Project goal

This repository exists to capture practical constraints and working patterns for Rust GPU compute, and to make it easier to reason about when Rust-on-GPU approaches are a good fit.
