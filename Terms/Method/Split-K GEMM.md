```json
{
  "term": "Split-K",
  "full_form": "Split-K GEMM",
  "folder": "Method",
  "definition": "把 GEMM 的 K 维切给多个 CTA 并行累加的策略",
  "tags": ["gemm", "gpu", "parallelism", "numerics"],
  "aliases": ["Split-K GEMM", "K 维切分", "split-KV"],
  "storage_path": "Terms/Method/Split-K GEMM.md"
}
```

## 📖 详细定义

GEMM 的并行通常切 M / N。但 decode 阶段常常是「M 很小（batch 小）、N 中等、K 很大（hidden dim）」，只切 M/N 会让大量 SM 空转。Split-K 于是把 K 也切开：每个 CTA 算一段部分和，最后再 reduce（atomic add 或单独 reduce kernel）。

收益：小 batch 下显著提高 SM 占用率，是 decode GEMM 的常用优化。

代价有三个，第三个最要命：

1. 额外的部分和写回与规约开销；
2. 需要额外显存或原子操作；
3. **切分数通常由启发式按 shape 决定，于是切分数随 batch 变**——部分和的合并顺序随之改变，浮点结果不再逐位一致。这是 batch 相关非确定性的头号来源。

确定性做法：K 不切（split-K = 1），或改用定点整数累加让顺序无关。

## 实际应用

- 主流推理框架的 GEMM 后端默认开 split-K；开 batch-invariant 时会显式禁用。DeepGEMM 的 deterministic 模式就是「K 永不切分」，并顺带绕开 cuBLASLt（其 heuristics 同样随 shape 变）。
- FlashMLA 的 `enable_batch_invariant` 是同一思路在 attention 上的版本：关掉 split-KV decode，让结果不依赖 batch 怎么划分。
- 何时不用：大 batch 的 prefill（M 已经够大，SM 已饱和），split-K 只有开销没有收益——所以它对 decode 更重要。
- 与投机解码的关系：投机让每步 query 数变多（验证多个 draft token），M 变大，split-K 的收益下降，切分数也跟着变——又一处不确定性来源。

## 🔬 前沿研究（3 项）

- `[2018] CUTLASS: Fast Linear Algebra in CUDA C++`：把 split-K 作为标准 GEMM 并行策略之一系统化实现（检索关键词：CUTLASS split-K GEMM）。
- `[2025] DeepGEMM`：面向 Hopper / Blackwell 的 FP8 GEMM 库，提供 `use_deterministic_algorithms` 开关，明确「deterministic 即 batch invariant，K 不切分」（检索关键词：DeepGEMM deterministic algorithms）。
- `[2025] Defeating Nondeterminism in LLM Inference`：把 split-K 归为 batch 相关非确定性的主要来源，并给出「固定切分」的解决范式（检索关键词：split-K nondeterminism inference）。

## 🔗 相关概念（3 个）

- `Batch Invariance`：目标——禁用 split-K 是达成它的手段之一。
- `Reduction Order`：机制——split-K 改变的正是归约顺序。
- `Atomic Add`：实现方式——split-K 常用原子加合并部分和，天然无序，因此也是不确定性的直接载体。

## ⚖️ 优缺点

- **优点**：① 小 batch / 瘦长矩阵下显著提升 SM 利用率，decode 阶段收益明显；② 实现相对简单，几乎所有 GEMM 库都支持。
- **缺点**：① 破坏逐位确定性，是 batch invariance 的主要障碍；② 额外规约开销，大 batch 下反而变慢。
