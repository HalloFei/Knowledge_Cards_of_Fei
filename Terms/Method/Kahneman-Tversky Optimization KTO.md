---
term: KTO
full_form: Kahneman-Tversky Optimization
folder: Method
definition: 基于前景理论，仅用单条好/坏标注即可对齐语言模型的偏好优化方法
tags: [alignment, llm, preference-learning, prospect-theory]
storage_path: KB/Method/kto.md
---

## 📖 详细定义
- KTO 借鉴行为经济学中 Kahneman-Tversky 前景理论的损失厌恶特性，将人类对"好输出"和"坏输出"的非对称感知建模到损失函数中。与 DPO/IPO 需要成对偏好数据不同，KTO 只需为每条输出标注"好"或"坏"（pointwise），大幅降低数据收集成本。

## 实际应用
- 适用场景：偏好对难以构造（如开放式生成、创意写作）或标注预算有限时，KTO 是首选轻量对齐方案。
- 在 Contextual AI 的实验中，KTO 在多个基准上匹配甚至超越 DPO，且标注成本减半。
- 工业界关注度上升：适合 UGC 场景中利用用户隐式反馈（点赞/踩）直接训练，无需配对。

## 🔬 前沿研究（2 项）
- `[2024] KTO: Model Alignment as Prospect Theoretic Optimization`：原始论文，证明 pointwise 信号在理论上足以恢复偏好排序（检索关键词：KTO prospect theory alignment Ethayarajh）。
- `[2024] ORPO: Monolithic Preference Optimization without Reference Model`：与 KTO 类似追求简化，将 SFT 和偏好优化合并为单阶段训练（检索关键词：ORPO monolithic preference optimization）。

## 🔗 相关概念（3 个）
- `DPO`：互补/竞争——DPO 需成对偏好数据，KTO 只需 pointwise 标注；数据充足时 DPO 更精确。
- `IPO`：同属 DPO 改进家族，IPO 改损失函数缓解过拟合，KTO 改数据需求降低标注成本。
- `RLHF`：祖父概念——KTO 是 RLHF 简化链条的最新环节：RLHF → DPO → KTO。

## ⚖️ 优缺点
- **优点**：1. 只需 pointwise 好/坏标注，数据收集成本远低于 pairwise 方法；2. 损失厌恶建模更贴合人类真实偏好心理。
- **缺点**：1. 缺少直接的偏好对比信号，细粒度排序能力弱于 DPO；2. 前景理论超参（λ 损失厌恶系数）需根据任务调整。