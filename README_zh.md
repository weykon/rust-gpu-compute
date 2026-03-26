# Rust GPU Compute Guide

一个面向 Claude Code 的插件与 marketplace 入口，用来帮助开发者判断 Rust GPU compute 的 trade-off、适用场景，以及 GPU 执行模型仍然带来的约束。

这个仓库更像一份定性判断指南，不是 benchmark 仓库。里面的建议主要帮助你判断 workload fit、execution model 与 memory behavior；真正的 break-even point 仍然取决于硬件、backend、runtime，以及数据是否已经驻留在 GPU 上。

## 适用范围

这个仓库主要帮助你做定性判断：

- 一个 Rust workload 在一开始是不是就适合放到 GPU
- 即使用 Rust 风格 API 表达 GPU 工作，哪些 GPU execution model 约束依然存在
- 在把工作迁到 GPU 之前，应该怎样检查 data layout、transfer boundary 与 branch behavior

它**不主张**这些事情：

- 给所有硬件环境都成立的 benchmark 式 break-even threshold
- 把 shader programming 说成已经被统一替代
- 证明所有 Rust concurrency pattern 都能高效映射到 GPU execution

## 主要覆盖内容

- 分支分化（branch divergence）与 warp 行为
- CPU↔GPU 传输成本
- Rust 在 GPU 端的常见限制
- memory coalescing 与 buffer 布局
- CPU / GPU 场景选择
- boid、粒子系统、图像处理、仿真等 workload 的判断方式
- VectorWare 风格 runtime 与传统 WGSL 工作流的区别
- runtime ergonomics、shader authoring 与 compilation path 之间的边界

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
