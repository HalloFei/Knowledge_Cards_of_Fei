{
  "term": "PTQ",
  "full_form": "Post Training Quantization",
  "folder": "Method",
  "definition": "训练后量化：无需重训练即可压缩模型的技术",
  "tags": ["quantization", "model-compression", "deployment", "optimization"],
  "storage_path": "Terms/Method/Post Training Quantization PTQ.md"
}

## 📖 详细定义
PTQ 是一种模型压缩技术，在模型训练完成后，通过降低权重和激活值的数值精度（如从 FP32 降至 INT8），减小模型体积和推理延迟，而无需访问训练数据或重新训练模型。与 QAT（Quantization Aware Training）相比，PTQ 更易实施但精度损失可能略大。

## 实际应用
PTQ 广泛应用于工业级模型部署场景，是当前 AI 推理优化的主流方案之一。主流框架（如 TensorRT、ONNX Runtime、PyTorch）均原生支持 PTQ。与 QAT 相比，PTQ 更适合快速迭代场景，但对精度敏感的任务（如大规模语言模型）仍面临挑战，混合精度量化（如 FP8/INT8）正在成为新趋势。

## 🔬 前沿研究
- [2024] AWQ: Activation-aware Weight Quantization for LLM：提出基于激活重要性的权重量化方法，显著提升大模型量化后性能
- [2023] GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers：首个高效准确的 LLM 量化方法，支持 4bit 量化

## 🔗 相关概念
- `QAT`：量化感知训练，在训练过程中模拟量化误差，精度更高但需要重训练
- `Pruning`：剪枝，通过移除冗余参数压缩模型，与量化互补
- `Distillation`：知识蒸馏，通过教师-学生网络压缩模型，可与量化联合使用

## ⚖️ 优缺点
- **优点**：
  1. 无需重训练，实施成本低、速度快
  2. 显著减小模型体积（通常压缩 4 倍）并加速推理
- **缺点**：
  1. 对低比特量化（如 4bit 以下）精度损失较大
  2. 对模型结构和超参数敏感，需调参以平衡精度与压缩率