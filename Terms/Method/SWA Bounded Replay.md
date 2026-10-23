```json
{
  "term": "SWA Bounded Replay",
  "full_form": "Sliding Window Attention Bounded Replay",
  "folder": "Method",
  "definition": "只回放末尾窗口重建 SWA KV、跳过整段历史的 prefill 优化",
  "tags": ["llm-inference", "attention", "kv-cache", "prefill"],
  "aliases": ["有界回放", "SWA 有界重放", "consumer compaction"],
  "storage_path": "Terms/Method/SWA Bounded Replay.md"
}
```

## 📖 详细定义

SWA（Sliding Window Attention）下每个 query 只看最近 W 个 key，所以 SWA 层的 KV 天然是 O(W) 的环形缓冲，而不是 O(L)。问题出在**恢复**：前缀缓存命中时，如果 SWA KV 没落盘，按常规做法要么整段重算，要么回退到某个 checkpoint 精确重算——两者都是 O(L)。

Bounded replay 承认一个事实：位置 i 的 query 只需要 `[i-W+1, i]` 的 key。所以只回放**末尾 K 个 token**（K ≥ W + 回退预算），并把这 K 个 token 的可见域裁成「原始滑窗 ∩ 保留矩形」：

```
abs_lo = clamp(max(pos - W + 1, rect_lo, valid_start), 0, pos + 1)
rect_lo = data_len - keep_tokens
```

回放起点之前的 token 既不进 attention，也不进缓存。于是 prefill 成本从 O(L) 变成 O(K)，与上下文长度无关。

**精确性边界（最容易被忽略的一点）**：保留区内的 attention 与全量计算逐位一致；但回放起点左侧的上下文被砍掉了，因此最老约一个窗口的 KV 相对「全量重算」是**近似**的。这是有意的近似、不是 bug——decode 只回看 W，过期即无用。反过来，`mask` 与 `compact` 两种实现的逐位等价，只证明「删行记账」正确，不证明截断无损。

两个实现形态：`mask` 保留全部 query 行、只收窄可见域（**不省算力**，纯参照物）；`compact` 真把截断区的 query 行删掉（省算力，要重写 varlen / 可见域 pivots / 写槽 / 压缩区间 / 采样行映射）。

## 实际应用

- DeepInfer（DeepSeek V4.1）拿它做 decoder 侧 prefill 截断：默认 `keep_tokens=140`（window 128 + 投机回退 6 + 余量），官方说法是总 prefill 计算「接近减半」。代价是前缀缓存命中上限变成 `L-134`，且 consumer 层 SWA 干脆不落盘。
- 成立前提很苛刻：模型必须是「后段层只维护 SWA、block KV 可由前段 hidden 一次性投影」的结构（YOCO consumer 层）。全层全注意力的通用模型（Llama/Qwen）用不上。
- 竞品路线是「精确」：vLLM 的 SWA + APC、SGLang 的 radix cache + HiCache，都在「存下来」和「精确重算」之间取舍；bounded replay 是第三个选项——**有界近似重算**。
- 是否主流：尚不是。但随着长上下文模型普遍混用 SWA / 线性 / 稀疏注意力，「历史可以丢」的假设会越来越常见，这类取舍会从特例变成常规设计。

## 🔬 前沿研究（3 项）

- `[2024] You Only Cache Once: Decoder-Decoder Architectures for Language Models (arXiv:2405.05254)`：把全局 KV 只算一次，后段层通过 cross-attention 复用——bounded replay 的结构前提（检索关键词：YOCO decoder-decoder KV cache）。
- `[2025] Native Sparse Attention (arXiv:2502.11089)`：可训练的、硬件对齐的块稀疏注意力；与 SWA 同属「限制可见域换成本」，区别是可见域由学习得到而非固定窗口（检索关键词：native sparse attention trainable）。
- `[2026] DeepSeek-V4 技术报告`（本工作区 Papers/ 下）：CED 架构把 SWA / 压缩 / 稀疏注意力组合在一起，bounded replay 是它在推理框架侧的配套优化（检索关键词：DeepSeek V4 CED hybrid attention）。

## 🔗 相关概念（3 个）

- `YOCO`：结构前提——只有「后段 KV 可由前段投影重建」，砍掉历史才不会丢 block KV。
- `PagedAttention`：互补而非替代——block KV 仍然分页存储并走前缀缓存，bounded replay 只动 SWA 那一半。
- `Speculative Decoding`：约束来源——回退预算决定 K 的下界；K 太小，draft 被拒需要回退时窗口就不够了。

## ⚖️ 优缺点

- **优点**：① 把 decoder 半区 prefill 从 O(L) 压到 O(常数)，上下文越长收益越大；② 让 consumer SWA 不必落盘，省下存储容量与 P→D 传输带宽。
- **缺点**：① 有界近似：最早一个窗口的 KV 与全量重算不一致，且缓存命中长度被砍掉 K 个 token；② 与 prompt_logprobs、consumer 层 Engram、跨边界的 DFlash target 等一堆功能互斥，CP 下还要「先 compact 再 CP」，实现复杂度很高。
