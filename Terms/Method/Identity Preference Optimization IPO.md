---
term: IPO
full_form: Identity Preference Optimization
folder: Method
definition: 用恒等映射替代 DPO 的 log-sigmoid 损失以缓解偏好过拟合的对齐方法
tags: [alignment, llm, preference-learning]
storage_path: KB/Method/ipo.md
---

## 📖 详细定义
- IPO 指出 DPO 隐含地假设偏好数据由确定性 Bradley-Terry 模型生成，当数据含噪或偏好标签非确定性时易过拟合。IPO 将损失函数中的 log-sigmoid 替换为恒等映射（identity），直接回归偏好对的对数概率差到固定 margin，从而实现更鲁棒的策略优化。

## 实际应用
- 当偏好标注一致性较低（如众包数据）时，IPO 比 DPO 更稳定，已在部分开源模型对齐实验中作为 DPO 的替代基线。
- 实际工业部署中使用较少，多见于学术对比实验；但其思想影响了后续 SimPO、KTO 等方法的设计。
- 适合场景：偏好数据噪声大、标注者间一致性低的中小规模对齐任务。

## 🔬 前沿研究（2 项）
- `[2024] SimPO: Simple Preference Optimization with a Reference-Free Reward`：去掉参考模型，用序列平均 log 概率作为隐式奖励，进一步简化 IPO/DPO 流程（检索关键词：SimPO reference-free preference）。
- `[2023] A General Theoretical Paradigm to Understand Learning from Human Feedback`：统一 DPO/IPO/KTO 的理论框架，分析不同损失函数对过拟合和分布偏移的敏感性（检索关键词：unified theory RLHF DPO IPO）。

## 🔗 相关概念（3 个）
- `DPO`：父概念——IPO 是对 DPO 损失函数的改进，针对其过拟合问题提出修正。
- `KTO`：互补——KTO 从前景理论出发只需单条标注，IPO 仍需偏好对但更鲁棒。
- `RLHF`：祖父概念——IPO/DPO 均属于绕开显式 RL 的 RLHF 简化方案。

## ⚖️ 优缺点
- **优点**：1. 对噪声偏好数据更鲁棒，不易过拟合；2. 实现与 DPO 同样简单，仅改损失函数。
- **缺点**：1. margin 超参需要调整，选择不当时性能不如 DPO；2. 在高质量偏好数据上收益有限，可能不如 DPO 拟合充分。