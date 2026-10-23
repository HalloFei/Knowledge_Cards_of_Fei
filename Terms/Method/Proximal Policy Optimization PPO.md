---
term: PPO
full_form: Proximal Policy Optimization
folder: Method
definition: 通过裁剪策略比率限制更新幅度的强化学习策略梯度算法
tags: [rl, policy-gradient, llm, alignment]
storage_path: KB/Method/ppo.md
---

## 📖 详细定义
- PPO 是一种 on-policy 策略梯度算法，核心思想是在每次更新时裁剪（clip）新旧策略的概率比率，将策略变化约束在信赖域内，从而在保持训练稳定的同时获得接近 TRPO 的性能，且实现远比 TRPO 简单。

## 实际应用
- 当前主流：PPO 是 RLHF 流程中训练语言模型的标准 RL 算法（InstructGPT、ChatGPT 均采用）。
- 游戏 AI 领域广泛使用：OpenAI Five（Dota2）、机器人控制（MuJoCo）等均以 PPO 为默认基线。
- 趋势：在 LLM 对齐场景中逐渐被 DPO/GRPO 等更简单方法替代，但在需要在线探索的复杂任务中仍不可取代。

## 🔬 前沿研究（2 项）
- `[2024] GRPO: Group Relative Policy Optimization`：DeepSeek 提出的 PPO 变体，去掉 critic 网络，用组内相对奖励做基线估计（检索关键词：GRPO DeepSeek group relative）。
- `[2023] ReMax: Reward-Maximizing PPO for RLHF`：改进 PPO 在 RLHF 中的奖励利用效率，减少样本浪费（检索关键词：ReMax RLHF reward maximizing）。

## 🔗 相关概念（3 个）
- `TRPO`：父概念——PPO 是 TRPO 的简化近似，用 clip 代替 KL 硬约束，训练更高效。
- `DPO`：互补/竞争——DPO 绕开 PPO 直接优化偏好，适合离线场景；PPO 保留在线探索能力。
- `Actor-Critic`：PPO 属于 Actor-Critic 框架，使用价值网络（Critic）估计优势函数以降低方差。

## ⚖️ 优缺点
- **优点**：1. 实现简单，clip 机制保证训练稳定性；2. 支持在线探索，适合复杂奖励信号场景。
- **缺点**：1. 需要同时维护策略网络和价值网络，显存开销大；2. 超参敏感（clip 范围、KL 惩罚系数等），调参成本高。