# Rust GPU Compute Guide

> 用 Rust 思维写 GPU 计算 — 避免常见陷阱，发挥 GPU 并行优势

## 背景

本项目源于对 [VectorWare](https://www.vectorware.com/blog/threads-on-gpu/) 的研究。VectorWare 实现了将 Rust `std::thread` 映射到 GPU warp 执行，让 Rust 代码可以直接在 GPU 上运行。

## 核心约束

GPU 计算有别于 CPU，计算时必须考虑:

1. **分支分化** — warp 内所有 lane 必须同步，分支导致等待
2. **传输成本** — CPU↔GPU 数据搬运耗时，细数据不值得搬
3. **Rust 限制** — 无 heap allocation、无递归、无 unwrap()/panic!
4. **内存合并** — 相邻线程访问相邻地址，否则带宽利用率低

详细说明见 [SKILL.md](SKILL.md)

## 适用场景

- ✅ 大数组并行变换 (map/reduce/filter)
- ✅ 图像处理、滤镜
- ✅ 科学计算 (Monte Carlo、物理仿真)
- ✅ 金融计算 (期权定价、风险分析)
- ✅ ML Inference
- ❌ 分支密集型算法
- ❌ < 1MB 小数据

## Boid 群体模拟

如果你想用 Rust 写 boid / 粒子系统 GPU 版本:

```rust
// GPU 不友好的写法 (嵌套循环、分支多)
fn update_boid(boid: &mut Boid, neighbors: &[Boid]) {
    for neighbor in neighbors {
        if distance(boid, neighbor) < RANGE {
            boid.velocity += separation(boid, neighbor);
        }
    }
}

// GPU 友好的写法:
// - 用空间哈希代替全量遍历
// - 用原子操作代替条件锁
// - 用 parallel reduction 代替串行求和
```

## 工具链

- [wgpu](https://github.com/gfx-rs/wgpu) — Rust GPU 库 (compute + graphics)
- [rust-gpu](https://github.com/EmbarkStudios/rust-gpu) — Rust → SPIR-V 编译器
- [naga](https://github.com/gfx-rs/naga) — Shader 语言翻译器

## 未来目标

1. **Rust → WGSL Transpiler** — 用 Rust 语法写 shader，直接产出 WGSL
2. **最佳实践文档** — GPU compute 开发模式库
3. **工具集** — 常见 GPU 算法的 Rust 实现

## 相关项目

- [VectorWare](https://www.vectorware.com/blog/threads-on-gpu/) — Rust thread on GPU
- [wgpu](https://github.com/gfx-rs/wgpu) — Rust GPU 图形和计算
- [rust-gpu](https://github.com/EmbarkStudios/rust-gpu) — Rust 写 GPU shader

## 许可证

MIT
