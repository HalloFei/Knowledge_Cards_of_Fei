```json
{
  "term": "Logits",
  "full_form": "Logits",
  "folder": "Concept",
  "definition": "模型输出层经softmax归一化前的原始未归一化分数向量",
  "tags": ["deep-learning", "llm", "probability", "neural-network"],
  "storage_path": "Terms/Concept/Logits.md"
}
```

## 📖 详细定义
- Logits 是神经网络最后一层线性变换的原始输出值（未经 softmax/sigmoid 归一化），取值范围为 (-∞, +∞)。在分类任务中，每个类别对应一个 logit 值，经 softmax 后转化为概率分布。该术语源自统计学中的 log-odds（对数几率）。

## 实际应用
- 在当前 LLM 推理管线中，logits 是核心中间产物：解码策略（greedy、top-k、top-p、temperature scaling）均直接操作 logits。知识蒸馏（KD）中教师模型的 logits 作为软标签传递给学生模型，是 2024-2026 年模型压缩的主流方案。此外，RLHF/DPO 等对齐方法也需要访问 policy 模型的 logits 来计算对数概率。logits 概念在深度学习中无处不在，不存在被淘汰的风险。

## 🔬 前沿研究（2 项）
- `[2025] Logit-based Distillation for Reasoning Models`：利用推理模型的 logits 分布蒸馏小模型，保留 CoT 推理能力（检索关键词：logit distillation reasoning LLM）。
- `[2024] Logits Cannot Lie: Faithfulness via Logit-Level Probing`：通过探测 logits 空间检测模型幻觉与不忠实输出（检索关键词：logit probing hallucination detection）。

## 🔗 相关概念（3 个）
- `Softmax`：互补关系——softmax 将 logits 映射为概率分布，是 logits 的下游归一化操作。
- `Cross-Entropy Loss`：互补关系——交叉熵损失直接作用于 logits（内部含 log-softmax），是训练时的核心损失函数。
- `Temperature`：互补关系——温度参数通过缩放 logits（logits/T）来控制输出分布的尖锐或平滑程度。

## ⚖️ 优缺点
- **优点**：① 保留了模型对各类别完整的置信度信息（信息无损）；② 支持灵活的后处理策略（温度调节、top-k 截断等）。
- **缺点**：① 原始 logits 不具备直观可解释性，需要归一化后才能作为概率理解；② 词表规模大时 logits 向量维度极高（如 LLM 词表 128K+），占用显存和带宽。