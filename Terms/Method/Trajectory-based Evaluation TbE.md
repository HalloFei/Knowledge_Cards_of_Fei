{
  "term": "轨迹导向评测",
  "full_form": "Trajectory-based Evaluation",
  "folder": "Method",
  "definition": "通过评估Agent推理与行动路径而非仅看最终结果的方法",
  "tags": ["evaluation", "agent", "reasoning"],
  "storage_path": "KB/Method/trajectory-based-evaluation.md"
}

📖 详细定义
轨迹导向评测是一种面向 AI Agent 的评测方法，关注 Agent 在完成任务过程中所经历的推理步骤、工具调用序列与决策路径（即"轨迹"），而不仅仅评判最终输出是否正确。它揭示 Agent 的规划能力、错误恢复能力和策略合理性。
🔬 前沿研究
[2023] AgentBench & Trajectory Analysis：系统性分析 Agent 轨迹中的失败模式，提出轨迹级评分框架，关键词：AgentBench trajectory。
[2024] LLM-as-Judge for Trajectories：利用 LLM 自动评估轨迹质量，替代人工标注，关键词：trajectory LLM judge。
🔗 相关概念
Outcome-based Evaluation：互补/对立——结果导向评测只看最终答案，轨迹导向评测关注过程，二者互补。
ReAct：关联——ReAct 框架产生的"思考-行动"序列本身就是轨迹，是轨迹评测的典型对象。
Process Reward Model (PRM)：子概念——PRM 对推理过程逐步打分，是轨迹评测在数学推理领域的具体实现。
⚖️ 优缺点
优点：1. 能识别"正确答案但错误推理"的问题，评估更深入；2. 可定位 Agent 失败的具体步骤，便于针对性改进。
缺点：1. 评估成本高，需逐步骤标注或设计自动评分机制；2. 轨迹最优性难以定义，不同策略可能殊途同归。