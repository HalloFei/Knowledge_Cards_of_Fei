{
  "term": "GEMM",
  "full_form": "General Matrix Multiplication",
  "folder": "Concept",
  "definition": "标准稠密矩阵乘加运算 D=αAB+βC,深度学习算力核心",
  "tags": ["llm", "gemm", "matrix", "inference"],
  "storage_path": "Terms/Concept/General Matrix Multiplication GEMM.md"
}
```

## 📖 详细定义
GEMM(General Matrix Multiplication)是 BLAS Level-3 中定义的稠密矩阵乘加运算 `D = α·A·B + β·C`,其中 A(M×K)、B(K×N)、C/D(M×N)。它是数值线性代数的基础操作,也是深度学习中 Transformer QKV 投影、FFN 升降维、卷积 im2col 展开等运算的数学本体。

## 实际应用
在 2026 年的 LLM 训练与推理中,GEMM 占 Transformer 计算量的 **90% 以上**,是绝对主流的算力形态。NVIDIA cuBLAS、CUTLASS、Meta FBGEMM、Apple Accelerate 等均围绕 GEMM 深度优化,Tensor Core、AMX、ARM SME 等硬件指令都专为其设计。近年 W4A16、INT8、FP8 量化 GEMM(如 Marlin、Machete)已成为大模型部署的事实标准;稀疏矩阵乘(SpMM)、分组 GEMM(Grouped GEMM,用于 MoE)是重要补充但尚未取代稠密 GEMM。

## 🔬 前沿研究
- `[2025] MpGEMM (ARM SME)`:在 Apple M4 Pro 上通过缓存感知分块和多向量加载,相比 Accelerate 平均加速 1.23×,验证了 SME 在 LLM 推理的潜力。
- `[2024] LRAMM`:结合低精度量化与随机 SVD(RSVD)的近似 GEMM,在可控误差内进一步压榨低比特矩阵乘性能。

## 🔗 相关概念
- `GEMV`:矩阵-向量乘,GEMM 在 N=1 时的退化形式,LLM 解码阶段(batch=1)主要瓶颈。
- `SpMM`:稀疏矩阵乘,GEMM 的稀疏变体,用于剪枝模型;与稠密 GEMM 互补。
- `GMM (Gaussian Mixture Model)`:高斯混合模型,与 GEMM 名字极易混淆,但属于统计学习,毫无关系。

## ⚖️ 优缺点
- **优点**:
  - 计算密集度高,能充分利用 Tensor Core / SIMD,峰值算力利用率可达 80%+。
  - 数学结构规整,便于分块、流水、量化与硬件加速。
- **缺点**:
  - 在解码(decoding)阶段退化为 GEMV,内存带宽受限,算力利用率骤降。
  - 对小 batch、不规则形状(skinny matrix)支持较差,需要特殊 kernel。