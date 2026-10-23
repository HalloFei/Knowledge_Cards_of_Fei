{
  "term": "FLOP",
  "full_form": "Floating Point Operation",
  "folder": "Concept",
  "definition": "计算机执行一次浮点数运算的基本单位",
  "tags": ["performance", "computing", "hardware"],
  "storage_path": "Concept/FLOP.md"
}

## 📖 详细定义
FLOP是衡量计算机计算能力的基本单位，指一次浮点数的加法、减法、乘法或除法运算。常用于评估处理器性能、对比不同硬件的计算效率，尤其在科学计算、深度学习训练等领域。

## 实际应用
目前仍是衡量AI训练和推理性能的核心指标之一。现代GPU/TPU的性能常用FLOPS（每秒浮点运算次数）表示，如A100峰值达312 TFLOPS。随着大模型兴起，计算成本评估、硬件选型都依赖FLOP指标，仍是主流方案。主要竞争指标包括INT8/INT4运算（量化场景下的替代指标）。

## 🔬 前沿研究
- `[2024] 混合精度训练优化`：通过降低精度（FP16/BF16）减少FLOP需求，同时保持模型精度。
- `[2023] FLOP-efficient架构设计`：研究如Mamba、线性Attention等低FLOP消耗的架构，挑战Transformer的计算效率瓶颈。

## 🔗 相关概念
- `FLOPS`：FLOP的速率单位（每秒浮点运算次数），衡量硬件性能。
- `MAC`：乘累加运算，深度学习中常用，1 MAC ≈ 2 FLOPs。
- `TOPS`：每秒万亿次运算，常用于量化推理场景（INT8精度）。

## ⚖️ 优缺点
- **优点**：标准化指标，便于跨硬件对比；直接反映计算密集型任务的成本。
- **缺点**：未考虑内存带宽、通信开销等瓶颈；不同精度下FLOP实际性能差异大。