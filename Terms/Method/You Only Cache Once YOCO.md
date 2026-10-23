```json
{
  "term": "YOCO",
  "full_form": "You Only Cache Once",
  "folder": "Method",
  "definition": "后段层复用前段 KV、全局只缓存一次的解码器架构",
  "tags": ["architecture", "attention", "kv-cache", "efficiency"],
  "aliases": ["只缓存一次架构", "decoder-decoder architecture"],
  "storage_path": "Terms/Method/You Only Cache Once YOCO.md"
}
```

## 📖 详细定义

标准 Transformer 每层都有自己的 KV cache，显存 O(L × layers)，长上下文下 KV 成为主要瓶颈。YOCO 把层切成两半：

- **self-decoder（前段）**：做全局注意力，产出「全局 KV」；
- **cross-decoder（后段）**：通过 cross-attention 复用前段产出的那一份 KV，自己不再新增全局 KV。

于是全局 KV 只存一份，显存从 O(L × layers) 降到 O(L)。

两个实现要点：① 前段用高效注意力（如 SWA）压成本，反正它只需要产出 KV；② **后段 KV 可以由前段最后一个 hidden 一次性投影出来**，不需要逐层顺序计算。

第二个要点是推理框架能真正吃到红利的地方：既然后段 KV 是投影，prefill 时就可以把后段的 query 行砍掉而不丢 KV（DeepInfer 的 consumer compaction 正是这么做的）。

边界：后段仍是 cross-attention，表达力与全注意力不完全等价，需要专门预训练；它不是「免费压缩」，而是把成本前移到了训练设计上。

## 实际应用

- DeepSeek V4.1 的 CED 架构采用该思路：`n_prefill_layer` 之前是 producer 层（KV 落盘），之后是 consumer 层（block KV 由边界 hidden 投影、只维护 SWA）；consumer 的 SWA KV 靠 bounded replay 重建，不落盘。
- 同类思路按「共享粒度」排队：MQA/GQA（共享 KV 头）→ CLA（相邻层共享）→ YOCO（全局共享一次）。粒度越粗，省得越多，表达力代价也越大。
- 是否主流：CLA / YOCO 在 2024-2026 的长上下文模型里逐渐增多，但通用推理框架支持仍弱——大多数框架假设「每层一个 KV cache」，这正是 DeepInfer 这类模型专用框架的机会。

## 🔬 前沿研究（3 项）

- `[2024] You Only Cache Once: Decoder-Decoder Architectures for Language Models (arXiv:2405.05254)`：提出 YOCO，把 KV cache 压到 O(L) 且保持长上下文能力（检索关键词：YOCO decoder-decoder）。
- `[2024] Reducing Transformer Key-Value Cache Size with Cross-Layer Attention (arXiv:2405.12981)`：CLA，相邻层共享 KV，处在 MQA 与 YOCO 之间的中间粒度（检索关键词：cross-layer attention KV sharing）。
- `[2026] DeepSeek-V4 技术报告`（本工作区 Papers/ 下）：CED 架构把 YOCO 与 SWA / 压缩 / 稀疏注意力组合，是该思路的规模化落地（检索关键词：DeepSeek V4 CED architecture）。

## 🔗 相关概念（3 个）

- `Multi-Query Attention`：共享粒度的另一极端——所有 head 共享一份 KV，但仍然每层一份，省的是 head 维度。
- `KV Cache`：被优化的对象；YOCO 改的是「层间是否共享」这一维。
- `SWA Bounded Replay`：下游推理优化——YOCO 让「历史可以丢」，bounded replay 才有可能成立。

## ⚖️ 优缺点

- **优点**：① KV 显存从 O(L × layers) 降到 O(L)，长上下文收益巨大；② 后段 KV 可一次性投影，prefill 少掉一大段顺序计算，给推理侧优化留出空间。
- **缺点**：① 表达力与训练复杂度有代价，需要专门的预训练方案；② 通用推理框架支持弱，往往要「模型 + 框架」co-design 才能真正吃到收益。
