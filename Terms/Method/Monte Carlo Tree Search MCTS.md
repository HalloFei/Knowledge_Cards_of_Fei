{
  "term": "MCTS",
  "full_form": "Monte Carlo Tree Search",
  "folder": "Method",
  "definition": "通过蒙特卡洛模拟构建搜索树以求解决策问题的启发式搜索算法",
  "tags": ["search", "decision-making", "game-ai"],
  "storage_path": "KB/Method/mcts.md"
}

📖 详细定义
MCTS 通过反复执行四步循环——选择（Selection）、扩展（Expansion）、模拟（Simulation）、回溯（Backpropagation）——来逐步构建一棵非对称搜索树，无需完整遍历状态空间即可逼近最优决策。在博弈论中属于零和博弈求解方法，在规划问题中属于 anytime 算法。

实际应用
- 博弈 AI（主流/成熟）：AlphaGo/AlphaZero 的核心搜索组件，结合深度神经网络的策略/价值估值，是围棋、国际象棋、德州扑克等棋牌类 AI 的标准范式，已完全取代传统 Minimax 在高复杂度博弈中的地位。
- LLM 推理增强（新兴/快速增长）：2023-2025 年最热门方向之一。LATS、SC-MCTS*、MCTSr 等工作将 MCTS 与大语言模型结合，用于数学推理（奥数级别）、代码生成、多步决策。LLaMA-3 8B + MCTS 在数学推理上可逼近 GPT-4 水平。目前属活跃研究方向，尚未成为生产标配，但势头强劲。
- 组合优化与调度（稳定应用）：用于 NP-hard 问题如作业车间调度（Job-Shop Scheduling）、旅行商问题、交通路由规划等，已有分布式并行 MCTS 方案。
- 机器人与自动驾驶（辅助方案）：用于多体动力学控制、路径规划等高维决策空间，通常与强化学习框架配合使用，属可选方案之一，非唯一主流。
- 竞争方案：在 LLM 推理场景中，Best-of-N 采样、Tree of Thoughts、Beam Search 等是更轻量的替代方案；在博弈领域 MCTS 仍无可替代。

🔬 前沿研究（2 项）
[2023] Language Agent Tree Search (LATS)：将 MCTS 与 LLM 结合用于多步推理与决策。检索关键词：LATS MCTS LLM。
[2024] MCTS in Mathematical Reasoning：用 MCTS 增强数学推理中的搜索与验证。检索关键词：MCTS math reasoning。

🔗 相关概念（3 个）
UCB（Upper Confidence Bound）：子概念；MCTS 中选择阶段的核心策略，通过 UCT 公式平衡利用与探索（exploitation vs. exploration）。
Minimax 搜索：易混淆；Minimax 需穷举博弈树至固定深度，MCTS 则通过随机模拟采样避免完整展开，在大状态空间中更具可行性。
强化学习（RL）：互补；MCTS 可作为 RL 中的策略改进组件（如 AlphaZero），RL 的价值网络反过来为 MCTS 提供更准确的节点评估，减少模拟次数。

⚖️ 优缺点
优点：① Anytime 特性，可随时中断并获得当前最优解；② 无需领域先验知识即可启动搜索。
缺点：① 模拟次数不足时估计偏差大；② 在高分支因子空间中收敛慢、计算开销大。