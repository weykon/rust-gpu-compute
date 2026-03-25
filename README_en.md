# Rust GPU Compute Guide

A Claude Code plugin and marketplace entry for reasoning about Rust GPU compute.

## What it helps with

- branch divergence and warp behavior
- CPU↔GPU transfer overhead
- Rust feature limits on the GPU
- memory coalescing and buffer layout
- CPU vs GPU decision-making
- boids, particle systems, image processing, and simulation workloads
- VectorWare-style runtime thinking vs traditional WGSL workflows

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
