```json
{
  "term": "Deterministic Sampling",
  "full_form": "Deterministic Sampling",
  "folder": "Method",
  "definition": "同种子同位置必产生同一 token 的无状态采样方法",
  "tags": ["sampling", "rng", "reproducibility", "llm-inference"],
  "aliases": ["确定性采样", "可复现采样", "deterministic inference"],
  "storage_path": "Terms/Method/Deterministic Sampling.md"
}
```

## 📖 详细定义

目标：`temperature > 0` 时，同一 prompt + 同一采样参数 → 同一输出 token 序列。注意它不等于贪婪解码（`temperature=0`）——随机性仍然保留，只是被变成**输入的函数**。

传统「每请求一条 RNG 流」的做法有两个漏洞：

1. **流的位置不可控**：抽到第几个随机数，取决于之前抽了几次；而投机解码每步接受几个 draft token 是可变的，于是采样位置漂移。
2. **CDF 累加顺序不可控**：浮点前缀和随 batch / tile 划分变化，同一个均匀随机数会落到不同区间。

无状态路线把两件事都消掉：随机数 = `f(seed, 绝对位置, 用途)`，用 counter-based RNG 直接算，不保存状态；CDF 用**量化后的整数**累加，顺序无关。

三个必备条件：

- 显式的 per-sequence seed（不能依赖全局 RNG）；
- batch-invariant 前向（否则 logits 都不一致）；
- 关闭会改变「每步验证几个 token」的变长投机（否则接受模式随 batch 变）。

## 实际应用

- DeepInfer：`SamplingParams.seq_seed` + 以绝对位置构造计数器（`position * 3 + case`，draft 用独立命名空间），CDF 定点化后两级搜索；要求 `DEEPINFER_BATCH_INVARIANT=1` 且 `DEEPINFER_ENABLE_MTP_VARLEN=0`。
- SGLang：`--enable-deterministic-inference` + `sampling_seed`（默认 42）；vLLM：per-request `seed` + 确定性算子。三家都收敛到「固定 seed + 确定性 kernel」这个组合。
- 刚需场景：GRPO 要「同 prompt 多个可复现样本」（用不同 seed 做多样性，而不是靠不可复现的随机）；评测可重跑；A/B 对比去噪；线上故障可复现。
- 代价：seed 的唯一性变成**调用方责任**，全量请求共用一个 seed 会得到「确定性重复」而不是多样性。

## 🔬 前沿研究（3 项）

- `[2011] Parallel Random Numbers: As Easy as 1, 2, 3 (Salmon et al., SC'11)`：Philox / Threefry 计数器型 RNG，让「按索引取随机数」在 GPU 上可并行且可复现，是这一路线的算法基础（检索关键词：counter-based RNG Philox）。
- `[2025] Defeating Nondeterminism in LLM Inference`：指出「采样器 RNG 与 batch 解耦」是端到端可复现的必要一环，而不仅是一个实现细节（检索关键词：deterministic sampling LLM inference）。
- `[2026] DeepInfer 融合采样器`：把 temperature / top-k / top-p / LSE / 采样 / 拒绝采样融进单 kernel，同时用定点算术保持可复现（本工作区可对照 TileLang fused_sample 实现）。

## 🔗 相关概念（3 个）

- `Batch Invariance`：前置条件——前向不一致，采样再确定也没有意义。
- `Counter-Based RNG`：实现载体——无状态随机数的来源，让「按位置取随机数」成为 O(1)。
- `Speculative Decoding`：冲突源——变长验证让「每步验证几个」依赖同批其他请求，必须固定长度才能完全确定。

## ⚖️ 优缺点

- **优点**：① 同输入同输出，评测 / RL / 回归链路可复现；② 随机性仍是输入的函数，可以枚举 seed 做可控的多样化采样。
- **缺点**：① 依赖 batch-invariant 与定长投机，吞吐有折损；② seed 管理下放给调用方，用错会产生「确定性重复」或跨请求相关性。
