---
term: DPO
full_form: Direct Preference Optimization
folder: Method
definition: 直接用偏好数据优化语言模型策略，无需训练奖励模型的对齐方法
tags: [alignment, llm, preference-learning]
storage_path: KB/Method/dpo.md
---

## 📖 详细定义
- DPO 将 RLHF 中的奖励建模 + PPO 两步流程简化为单步：利用 Bradley-Terry 偏好模型的闭式解，直接在偏好对（chosen/rejected）上对策略做极大似然优化，绕开了显式奖励函数和强化学习训练的不稳定性。

## 实际应用
- 当前主流：Llama 3、Zephyr、Mistral 等开源模型的对齐阶段大量采用 DPO 或其变体，已成为 RLHF 的轻量替代方案。
- 工业界（Anthropic、Meta）在人类偏好数据充足时优先尝试 DPO，因其实现简单、超参少、训练稳定。
- 局限：当偏好数据噪声大或分布偏移严重时，DPO 性能可能不如在线 RLHF。

## 🔬 前沿研究（2 项）
- `[2024] KTO: Model Alignment as Prospect Theoretic Optimization`：用 Kahneman-Tversky 前景理论替代 Bradley-Terry，只需单条好/坏标注即可训练（检索关键词：KTO alignment prospect theory）。
- `[2024] Self-Play Preference Optimization (SPPO)`：模型自博弈生成偏好对再做 DPO，减少对人类标注的依赖（检索关键词：self-play preference optimization）。

## 🔗 相关概念（3 个）
- `RLHF`：父概念——DPO 是 RLHF 的简化替代，省去奖励模型和 PPO 阶段。
- `PPO`：RLHF 中的策略优化算法；DPO 的核心贡献正是证明可以不用 PPO 达到等价效果。
- `IPO (Identity Preference Optimization)`：DPO 的改进变体，用恒等映射替代 log-sigmoid 损失以缓解过拟合。

## ⚖️ 优缺点
- **优点**：1. 无需训练奖励模型，流程简单且训练稳定；2. 超参数少，计算开销显著低于 PPO。
- **缺点**：1. 依赖离线偏好数据质量，对分布偏移敏感；2. 缺乏在线探索能力，可能收敛到次优策略。