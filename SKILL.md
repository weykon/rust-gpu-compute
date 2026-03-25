---
name: rust-gpu-compute
description: |
  Rust GPU Compute Development Guide — helps developers write GPU-accelerated Rust code with proper patterns and awareness of GPU-specific constraints. Use when user mentions: GPU compute, WGSL, compute shader, GPU acceleration, SIMD, warp, CUDA, parallel GPU, data parallel, rust gpu, vectorware, boid, particle system, graphics compute. Trigger especially when user is working on: data-parallel processing, scientific computing, image processing, ML inference, or any compute-heavy Rust code that could benefit from GPU acceleration.
---

# Rust GPU Compute Development Guide

## 项目背景

本项目基于 VectorWare 的创新实践：将 Rust `std::thread` 映射到 GPU warp 执行。

> **VectorWare 核心思路**: 在 GPU 上添加 Rust runtime，对外接口是 Rust CPU 代码，自动转译成 GPU 指令执行。

本 skill 旨在帮助开发者在 Rust GPU 计算场景下写出高效代码，避免常见陷阱。

---

## GPU 计算约束 — 核心注意事项

### 1. 分支分化 (Branch Divergence) — 最容易踩的坑

```
问题：if (thread_id == 0) { ... } 这类分支会导致 GPU 慢速
原因：warp 内所有 lane 必须同步执行，分支会让部分 lane 等待
```

**反面例子 (WGSL):**
```wgsl
// ❌ 很差 - 每个 lane 走不同路径，warp 要等待
if (local_id == 0u) {
    sum += values[i];
} else {
    sum -= values[i];
}
```

**正面例子 (数据并行):**
```wgsl
// ✅ 好 - 所有 lane 走相同路径，只是数据不同
sum += values[i];  // 全部 lane 并行执行
```

**Rust 思维转换:**
```rust
// Rust 习惯写法 ❌ GPU 不友好
if thread_id == 0 {
    do_something();
} else {
    do_other_thing();
}

// GPU 友好写法 ✅ - 用掩码或避免分支
// 方案 A：两个 kernel
// 方案 B：用 atomic 操作代替条件判断
```

### 2. 内存传输成本 (Memory Transfer Overhead)

```
问题：CPU ↔ GPU 数据搬运耗时，细数据不值得搬
经验法则：< 1MB 数据通常 CPU 直接算更快
```

**优化策略:**
```rust
// ❌ 频繁搬运
for batch in data.chunks(100) {
    gpu.upload(batch);      // 每次传输有固定开销
    gpu.run();
    gpu.download();
}

// ✅ 批量搬运
let all_data = gpu.upload一次性(data);  // 一次大传输
for _ in 0..iterations {
    gpu.run();  // 纯 GPU 计算，无传输
}
```

### 3. Rust 特性限制

```
不支持的特性:
- ❌ 堆分配 (heap allocation)
- ❌ 递归 (recursion)
- ❌ 动态大小类型 ( DST )
- ❌ unwrap()/panic! 在 GPU 端
```

**可行替代:**
```rust
// ❌ Rust 习惯
let result = vec![0u32; n];  // 堆分配

// ✅ GPU 友好
let mut buffer = Buffer::zeroed(n);  // 预分配，栈上或固定内存
```

### 4. 内存合并 (Memory Coalescing)

```
问题：线程访问模式不 coalescing 会导致内存带宽利用率低
原则：相邻线程访问相邻内存地址
```

**反面例子:**
```rust
// ❌ 差 - 线程 i 访问 threads[i*stride] 步长访问
for i in 0..N {
    let val = data[i * STRIDE];
}
```

**正面例子:**
```rust
// ✅ 好 - 线程 i 访问 data[i]，连续访问
for i in 0..N {
    let val = data[global_id()];  // coalesced
}
```

### 5. GPU vs CPU 场景选择

| 场景 | 推荐 |
|------|------|
| 大数组并行变换 (map/reduce) | ✅ GPU |
| 分支多的算法 | ❌ CPU |
| < 1MB 数据 | ❌ CPU |
| 图像处理/滤镜 | ✅ GPU |
| Monte Carlo 仿真 | ✅ GPU |
| 加密/哈希 | ✅ GPU |
| 排序/搜索 | ⚠️ 视情况 |

---

## VectorWare 运行时模型 vs 传统 WGSL

```
传统 WGSL 方式:
  你写 WGSL 代码 → 直接 GPU 执行
  需要学习 GPU 思维和 WGSL 语法

VectorWare 方式:
  你写 Rust thread → runtime 转译 → GPU warp 执行
  可以用 Rust 思维写 GPU 代码，但有限制

Rust → WGSL Transpiler (理想目标):
  你写 Rust 代码 → 转译成 WGSL → GPU 执行
  保留 Rust 语法，完全控制 GPU
```

---

## Boid 群体模拟案例分析

**Boid 核心算法约束分析:**

```rust
// Boid 规则通常包含:
fn update_boid(boid: &mut BoiId, neighbors: &[Boid]) {
    // 规则1: 分离 (Separation) - 需要遍历邻居
    for neighbor in neighbors {  // ❌ 嵌套循环，GPU 难优化
        let diff = boid.pos - neighbor.pos;
        boid.vel += diff * SEPARATION权重;
    }

    // 规则2: 对齐 (Alignment) - 平均速度
    let avg_vel = 计算平均速度(neighbors);  // ⚠️ 需要归约操作

    // 规则3: 内聚 (Cohesion) - 移向中心
    let center = 计算邻居中心();  // ⚠️ 归约 + 广播
}
```

**GPU 优化思路:**

| 问题 | 传统 Rust | GPU 友好版本 |
|------|----------|------------|
| 邻居遍历 | 嵌套循环 | 空间哈希网格化 |
| 平均计算 | for loop | parallel reduction |
| 条件判断 | if (dist < RANGE) | 用 0*操作代替 |

---

## 开发检查清单

写 GPU compute 代码前，逐项检查:

- [ ] 数据大小 > 1MB？
- [ ] 无嵌套分支？
- [ ] 无递归？
- [ ] 无 heap allocation？
- [ ] 内存访问模式 coalesced？
- [ ] 相邻线程访问相邻地址？
- [ ] 批量传输代替频繁小传输？

---

## 参考资源

- [VectorWare: Threads on GPU](https://www.vectorware.com/blog/threads-on-gpu/) — Rust thread 映射到 GPU warp
- [wgpu](https://github.com/gfx-rs/wgpu) — Rust GPU 图形和计算库
- [rust-gpu](https://github.com/EmbarkStudios/rust-gpu) — Rust 到 SPIR-V 编译器
- [WGSL Spec](https://www.w3.org/TR/WGSL/) — WebGPU Shading Language

---

## 项目目标

本项目旨在:

1. **Capturing 经验** — 记录 GPU 计算开发的 constraints 和最佳实践
2. **工具化** — 未来实现 Rust → WGSL Transpiler
3. **生态** — 为 Rust GPU 计算提供开发指南和工具集

如果你想用 Rust 写 boid、粒子系统、数据并行处理，本 guide 帮你避免常见陷阱。
