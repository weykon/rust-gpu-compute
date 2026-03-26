# Rust GPU Compute Guide

A Claude Code plugin and marketplace entry for reasoning about Rust GPU compute trade-offs.

This repository is a qualitative guide, not a benchmark suite. The guidance here is meant to help developers reason about workload fit, execution-model constraints, and memory behavior; exact break-even points depend on hardware, backend, runtime, and whether data already resides on the GPU.

## Scope

This repository helps with qualitative reasoning about:

- when a Rust workload is GPU-friendly in the first place
- which GPU execution-model constraints still matter even behind a Rust-flavored API
- how to review data layout, transfer boundaries, and branch behavior before moving work to the GPU

It does **not** claim to provide:

- benchmark-backed break-even thresholds for every hardware setup
- a universal replacement for shader programming
- proof that every Rust concurrency pattern maps efficiently to GPU execution

## What it helps with

- branch divergence and warp behavior
- CPU↔GPU transfer overhead
- Rust feature limits on the GPU
- memory coalescing and buffer layout
- CPU vs GPU decision-making
- boids, particle systems, image processing, and simulation workloads
- VectorWare-style runtime thinking vs traditional WGSL workflows
- clearer boundaries between runtime ergonomics, shader authoring, and compilation paths

## Repository structure

- `.claude-plugin/marketplace.json` — canonical marketplace manifest
- `.claude-plugin/plugin.json` — plugin manifest
- `skills/guide/SKILL.md` — canonical skill file

## Install from GitHub marketplace source

```text
/plugin marketplace add weykon/rust-gpu-compute
/plugin install rust-gpu-compute@rust-gpu-compute-guide
```

## Local development

```bash
claude --plugin-dir /absolute/path/to/rust-gpu-compute
```

Then reload plugins if needed:

```text
/reload-plugins
```

## Skill command

```text
/rust-gpu-compute:guide
```

## Notes

- This repo intentionally keeps the plugin and the marketplace in the same repository.
- The canonical skill content is fully English.
- `README_zh.md` provides the Chinese version of the project introduction.

## Related projects

- [VectorWare](https://www.vectorware.com/blog/threads-on-gpu/)
- [wgpu](https://github.com/gfx-rs/wgpu)
- [rust-gpu](https://github.com/EmbarkStudios/rust-gpu)

## License

MIT
