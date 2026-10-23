```json
{
  "term": "投机解码",
  "full_form": "Speculative Decoding",
  "folder": "Method",
  "definition": "用小模型先猜多个 token、大模型一次并行验证的推理加速方法",
  "tags": ["inference", "llm", "decoding", "latency"],
  "aliases": ["投机采样", "speculative sampling", "预测解码"],
  "storage_path": "Terms/Method/投机解码 Speculative Decoding.md"
}
```

## 📖 详细定义

自回归解码的瓶颈在于：每生成一个 token，都要把整个模型跑一遍。GPU 的算力大量空闲，因为这一步是**访存受限**的——权重读一遍，只产出一个词。

投机解码把"生成"和"验证"拆开：先用一个小而快的草稿模型连续猜 γ 个 token，再让目标大模型**一次前向**同时给这 γ 个位置打分，然后从前往后逐个判定接受还是拒绝。

关键的接受规则（保证无损）：

- 第 i 个草稿 token 以概率 `min(1, p_target(x_i) / p_draft(x_i))` 被接受；
- 一旦拒绝，就从修正分布 `normalize(max(0, p_target − p_draft))` 重新采一个 token，后面的草稿全部丢弃；
- 全部接受，则额外从目标分布采一个 token。

这套拒绝采样（rejection sampling）让最终输出的分布**严格等于**目标模型自己的分布——所以它是无损加速，不是近似。这是它和"用蒸馏小模型直接替代"最本质的区别。

加速比大致取决于两件事：草稿的接受率 α，和草稿/验证的成本比。期望每轮产出 `(1 − α^(γ+1)) / (1 − α)` 个 token；α 越高、草稿越便宜，越划算。

常见变体：Medusa（在目标模型上加多个解码头，不用独立草稿模型）、EAGLE（在特征层做自回归草稿）、Lookahead Decoding、自投机（用模型自身浅层当草稿）、以及把输入里已有的片段直接当草稿（prompt lookup，代码编辑场景特别有效）。

## 实际应用

- 已经是主流推理栈的标配能力：vLLM、TensorRT-LLM、SGLang 都有实现，通常以"草稿模型陪跑"或"带 draft head"的形式开启。
- 收益强依赖任务：代码补全、模板化输出、格式化 JSON、摘要改写这类"高可预测"文本接受率高，常见 2–3 倍；开放域创作、强推理任务的接受率低，收益会掉到 1.2 倍上下甚至为负。
- 代价被低估的两处：一是显存（多一个草稿模型）；二是**大 batch 下收益衰减**——batch 大了以后 GPU 本来就吃满，投机带来的并行冗余反而变成负担，所以它更适合低延迟、小 batch 的在线场景，而不是吞吐优先的离线批处理。
- 竞争方案：量化、KV cache 优化（如共享缓存的架构）、连续批处理。它们和投机解码正交，可以叠加。

## 🔬 前沿研究（3 项）

- `[2023] Fast Inference from Transformers via Speculative Decoding (Leviathan et al., arXiv:2211.17192)`：提出"草稿—验证"框架并给出无损性的证明，是这条线的起点（检索关键词：speculative decoding draft verify lossless）。
- `[2023] Accelerating Large Language Model Decoding with Speculative Sampling (Chen et al., DeepMind, arXiv:2302.01318)`：同期独立工作，把接受/拒绝规则写成可证明保持分布的采样形式，2–2.5 倍加速（检索关键词：speculative sampling rejection sampling distribution preserving）。
- `[2024] EAGLE: Speculative Sampling Requires Rethinking Feature Uncertainty (arXiv:2401.15077)`：把草稿从 token 层移到特征层做自回归，显著提高接受率，是当前工程落地较多的一支（检索关键词：EAGLE feature-level draft speculative sampling）。

## 🔗 相关概念（3 个）

- `自回归解码（Autoregressive Decoding）`：被优化的对象——逐 token 生成，每步一次完整前向，受限于访存带宽而非算力。
- `拒绝采样（Rejection Sampling）`：正确性的来源——正是它保证"猜错的那些 token 不会污染输出分布"，去掉它投机解码就退化成有偏的近似。
- `Medusa / Lookahead Decoding`：同族方案——都不需要独立草稿模型，用多头或并行窗口制造候选，取舍在"实现简单"与"接受率"之间。

## ⚖️ 优缺点

- **优点**：① 输出分布与目标模型完全一致，不需要重新训练或微调，属于"免费"的延迟优化；② 与量化、KV cache 优化正交，可叠加。
- **缺点**：① 收益高度依赖任务与 batch size，预测不准时可能负收益；② 多一个模型/多一套调度，工程复杂度与显存开销都上去了，且只有在严格实现接受规则时才无损。
