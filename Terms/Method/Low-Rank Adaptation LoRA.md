{
  "term": "LoRA",
  "full_form": "Low-Rank Adaptation",
  "folder": "Method",
  "definition": "通过低秩矩阵分解实现大模型参数高效微调的方法",
  "tags": ["llm", "peft", "fine-tuning", "adaptation"],
  "storage_path": "Terms/Method/low-rank-adaptation-lora.md"
}
```

## 📖 详细定义
LoRA 冻结预训练模型的原始权重，在其旁路注入两个低秩矩阵 A、B（ΔW = BA），仅训练这两个小矩阵来近似全参数微调的效果，从而将可训练参数量降低数百至上千倍。

## 实际应用
截至 2026 年，LoRA 及其变体（QLoRA、DoRA、LoRA+）已成为开源社区微调 LLM 与扩散模型（Stable Diffusion）的**事实标准**。Hugging Face PEFT、Unsloth、Axolotl 等主流训练框架均以 LoRA 为核心。竞争方案包括 Prefix Tuning、IA³、Adapter，但因效果与工程便利性不及 LoRA 已逐渐边缘化；全量微调仅在追求极致效果的闭源大厂场景中使用。

## 🔬 前沿研究（2 项）
- `[2024] DoRA: Weight-Decomposed Low-Rank Adaptation`：将权重分解为幅度与方向，方向部分用 LoRA新，性能接近全参数微调。
- `[2024] LoRA+`：为 A、B 两矩阵设置不同学习率（B 更大），显著提升收敛速度与最终精度。

## 🔗 相关概念（3 个）
- `PEFT`：父概念，LoRA 是 PEFT 家族中最流行的具体方法。
- `QLoRA`：子概念，在 4-bit 量化基座上应用 LoRA，进一步降低显存开销。
- `Adapter`：互补/易混淆，同为参数高效微调，但 Adapter 串行插入层内会增加推理延迟，LoRA 可合并回权重零延迟。

## ⚖️ 优缺点
- **优点**：
  - 可训练参数极少（通常 <1%），显存占用低，便于消费级 GPU 微调。
  - 训练完成后可将 ΔW 合并到原权重，推理零额外开销。
- **缺点**：
  - 秩 r 的选择依赖经验，过小欠拟合、过大失去效率优势。
  - 在与预训练分布差异极大的任务上，效果仍逊于全参数微调。