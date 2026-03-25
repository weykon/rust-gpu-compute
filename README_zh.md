# Rust GPU Compute Guide

一个面向 Claude Code 的插件与 marketplace 入口，用来帮助开发者判断 Rust GPU compute 场景是否合适，以及应该怎样写得更像 GPU 代码。

## 主要覆盖内容

- 分支分化（branch divergence）与 warp 行为
- CPU↔GPU 传输成本
- Rust 在 GPU 端的常见限制
- memory coalescing 与 buffer 布局
- CPU / GPU 场景选择
- boid、粒子系统、图像处理、仿真等 workload 的判断方式
- VectorWare 风格 runtime 与传统 WGSL 工作流的区别

## 仓库结构

- `.claude-plugin/marketplace.json` — 正式 marketplace 清单
- `.claude-plugin/plugin.json` — plugin manifest
- `skills/guide/SKILL.md` — 正式 skill 文件

## 从 GitHub 安装

```text
/plugin marketplace add weykon/rust-gpu-compute
/plugin install rust-gpu-compute@rust-gpu-compute-guide
```

## 本地开发

```bash
claude --plugin-dir /absolute/path/to/rust-gpu-compute
```

如果需要，可在 Claude Code 里执行：

```text
/reload-plugins
```

## Skill 命令

```text
/rust-gpu-compute:guide
```

## 说明

- 这个仓库刻意把 plugin 和 marketplace 放在同一个 repo 里。
- 正式 skill 内容已经改成全英文。
- `README_en.md` 是英文介绍版本。

## 相关项目

- [VectorWare](https://www.vectorware.com/blog/threads-on-gpu/)
- [wgpu](https://github.com/gfx-rs/wgpu)
- [rust-gpu](https://github.com/EmbarkStudios/rust-gpu)

## License

MIT
