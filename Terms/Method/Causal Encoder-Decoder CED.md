```json
{
  "term": "CED",
  "full_form": "Causal Encoder-Decoder",
  "folder": "Method",
  "definition": "编码器只算一次、解码器 KV 由编码器隐状态投影的架构",
  "tags": ["architecture", "kv-cache", "moe", "long-context"],
  "aliases": ["因果编码器-解码器", "CED 架构", "causal encoder-decoder"],
  "storage_path": "Terms/Method/Causal Encoder-Decoder CED.md"
}
```

## 📖 详细定义

CED 是 DeepSeek-V4.1-Flash 的主干架构：一个 40 层 Transformer 被切成 **20 层 causal encoder + 20 层 decoder**。encoder 正常做因果注意力、产出隐状态；decoder **不再为每一层维护独立 KV**，而是从 encoder 最后一层隐状态**一次性投影**出「全局 KV」，各 decoder 层共享复用它。

这个「投影」是全部收益的来源：decoder 的 KV 不随层数线性增长，全局 KV 压到 **890 bytes/token**，于是 1M 上下文在显存上才成立。

**和标准 encoder-decoder（T5 / BART）的区别，就在「Causal」这个词上**：T5 的 encoder 是**双向**的，只把 source 编码一次、不需要 KV cache，decoder 靠 cross-attention 去读它。CED 的 encoder 是**因果**的，因为它要支持「单流自回归 + 前缀缓存」——encoder 自己也有 KV cache，只是 decoder 那一半不再重复增长。所以它不是「T5 式的编码-解码」，而是 YOCO 那一族「decoder-decoder」思路的因果化工程形态。

**激活参数不对称**也是设计的一部分：prefill 阶段每 token 只激活约 8B（主要来自 encoder），decode 阶段 16B（要逐 token 生成，MoE 更宽）。输入密集的 agentic 负载因此便宜，输出密集的对话负载收益小。

**命名对照（读代码时很容易迷路）**：同一个概念在三处叫法不同——模型卡叫 CED 的 encoder / decoder；DeepInfer 代码里叫 **producer / consumer**（YOCO 术语），分界是 `n_prefill_layer`（V4.1-Flash 为 20）；部署 Q&A 里叫 encoder / decoder layers。三者指同一刀切分。

## 实际应用

- DeepSeek-V4.1-Flash 的具体数字：552B 总参、40 层、1M 上下文、890 B/token 全局 KV（约为 V4-Flash 的 1/4，V1 的 1/437）；SWA 持久化 KV 约为 V4-Flash 的 1/8；45T token 预训练，稀疏注意力先在 64K 训练、34T token 处扩到 1M。
- 框架侧配套（DeepInfer）：decoder 层被当作 YOCO consumer 层，block KV 由边界 hidden 经 `shared_hidden_kv_proj` 投影写入；consumer 的 SWA KV **不落盘**，prefill 时用 SWA bounded replay 只回放末尾 140 个 token。
- 适用条件：超长上下文 + 输入密集（代码 agent、文档处理、多工具编排）。短 prompt 的交互式场景吃不到多少好处，还要付 encoder/decoder 切分的固定成本。
- 竞争路线要分清层次：CED 是**结构侧**解法（改模型本身）；MLA + 稀疏注意力（V3.2 系）、跨层 KV 共享（CLA）、线性/混合注意力是同类结构解法；分页 KV + 分层存储（vLLM / SGLang 的 HiCache）是**系统侧**解法。两者可以叠加，不互斥。

## 🔬 前沿研究（3 项）

- `[2024] You Only Cache Once: Decoder-Decoder Architectures for Language Models (arXiv:2405.05254)`：CED 的思想前身——把层分成「产 KV」和「用 KV」两半，全局只缓存一次（检索关键词：YOCO decoder-decoder）。
- `[2024] Reducing Transformer Key-Value Cache Size with Cross-Layer Attention (arXiv:2405.12981)`：相邻层共享 KV 的中间粒度方案，正好用来对照 CED 的「全局共享一次」走到了哪一步（检索关键词：cross-layer attention KV sharing）。
- `[2026] DeepSeek-V4.1-Flash 技术报告 / 模型卡`：CED 的官方定义，以及 890 B/token、8B/16B 激活、1/8 SWA 持久化等数据；推理侧实现可对照 DeepInfer 的 `doc/inference_deployment_qa.md`（检索关键词：DeepSeek V4.1 Flash CED）。

## 🔗 相关概念（3 个）

- `YOCO`：同族——CED 是 YOCO「decoder-decoder」思路在 DeepSeek 的因果化实现，区别在 encoder 侧还被 CSA2 压了一道。
- `CSA2 (Compressed Sparse Attention 2)`：CED 内部的注意力机制——Full / Reindex / Reuse 三种静态模式跨层共享 main KV 与 indexer K，把 encoder 侧的 KV 再压一层。
- `SWA Bounded Replay`：推理侧配套——CED 让 decoder 的 SWA 可以丢，bounded replay 负责用末尾 140 个 token 把它重建出来。

## ⚖️ 优缺点

- **优点**：① 全局 KV 不再随层数增长（890 B/token），1M 上下文才在显存与带宽上可行；② 激活参数按阶段不对称（prefill 8B / decode 16B），输入密集负载的单位推理成本大幅下降。
- **缺点**：① encoder 必须因果，放弃了双向上下文，且训练方案要重新设计；② decoder KV 由投影得到，表达力弱于逐层独立 KV，且强依赖框架配合（通用推理框架很难直接支持）。
