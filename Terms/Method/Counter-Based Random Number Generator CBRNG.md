```json
{
  "term": "CBRNG",
  "full_form": "Counter-Based Random Number Generator",
  "folder": "Method",
  "definition": "由种子与计数器经纯函数算出随机数、无内部状态的 RNG",
  "tags": ["rng", "parallel-computing", "determinism", "gpu"],
  "aliases": ["计数器型随机数发生器", "Philox", "Threefry"],
  "storage_path": "Terms/Method/Counter-Based Random Number Generator CBRNG.md"
}
```

## 📖 详细定义

传统 PRNG（Mersenne Twister、xorshift）是**有状态**的：抽一次，状态前进一次。并行时只有两条路——每线程一条流（几乎不可复现），或者加锁（慢）。

Counter-based RNG 改成纯函数：

```
random(n) = F(key, counter + n)
```

`F` 是一个块密码式的混合函数（Philox 用多轮 multiply-xor，Threefry 用加法-旋转），给定 `(key, counter)` 直接算出第 n 个随机数，不需要先算前面所有。

三个性质决定了它在推理系统里的价值：

- **可随机访问**：第 n 个随机数是 O(1)，不依赖历史；
- **与划分无关**：谁算、算几路、怎么切线程，结果都一样；
- **天然可复现**：只要 `(key, counter)` 相同，结果相同。

代价：单次生成比有状态 PRNG 贵（多轮乘法），但 GPU 上吞吐被并行度掩盖；统计质量需要用 TestU01 / BigCrush 一类套件验证。

## 实际应用

- 主流 GPU 栈：cuRAND 的 Philox、PyTorch 的 CUDA Generator、JAX 的 `jax.random` 都用这一族。JAX 更进一步，把 PRNG key 做成显式参数，让「随机性即纯函数」贯穿整个 API。
- LLM 推理里的用法：把 `(seq_seed, 绝对位置)` 当 `(key, counter)`，则同一 token 位置的随机数永远是同一个——无论 batch 怎么组、是否重算、投机接受了几步。
- 分布式契合度极高：数据并行各 rank 用不同 key，天然保证「不重不漏」，不需要同步 RNG 状态。
- 是否主流：在 GPU / 并行数值计算里已是事实标准；在推理框架的采样器里仍属较新实践。

## 🔬 前沿研究（3 项）

- `[2011] Parallel Random Numbers: As Easy as 1, 2, 3 (Salmon et al., SC'11)`：提出 Philox / Threefry 并证明其统计质量，成为 GPU 上的事实标准（检索关键词：Random123 Philox Threefry）。
- `[2018] JAX: composable transformations of Python+NumPy programs (arXiv:1806.00724)`：把 PRNG key 显式化、函数化，是「无状态随机」在框架层的完整实践（检索关键词：JAX explicit PRNG key）。
- `[2026] DeepInfer 采样器`：把 `(seq_seed, position, 用途)` 映射到计数器，draft 与 target 用不同命名空间，实现按位置可复现的采样（检索关键词：counter-based sampling LLM serving）。

## 🔗 相关概念（3 个）

- `Philox`：最常用的 counter-based 算法实现（10 轮 multiply-xor），cuRAND / PyTorch 的选择。
- `Mersenne Twister`：易混淆——经典**有状态** PRNG，周期极长但不可随机访问，不适合并行可复现场景。
- `Deterministic Sampling`：上游应用——用 CBRNG 把「抽第几个随机数」变成「这个位置对应哪个随机数」。

## ⚖️ 优缺点

- **优点**：① 可随机访问、天然并行、与线程/批划分无关；② 无状态，易序列化、易复现、易跨设备对齐。
- **缺点**：① 单次生成成本高于有状态 PRNG；② key 空间设计不当（重复 key）会引入相关性，需要谨慎分配。
