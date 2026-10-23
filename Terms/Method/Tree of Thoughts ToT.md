{
  "term": "ToT",
  "full_form": "Tree of Thoughts",
  "folder": "Method",
  "definition": "将推理空间组织为树结构并结合搜索与回溯的提示方法",
  "tags": ["prompting", "reasoning", "llm", "search"],
  "storage_path": "KB/Method/tree-of-thoughts.md"
}

📖 详细定义
Tree of Thoughts 将 Chain of Thought 的线性推理链扩展为树状结构：每个节点是一个"思维状态"（partial solution），通过分支生成多个候选下一步，再利用评估器对节点打分并选择最有前景的路径继续搜索，支持 BFS/DFS 及回溯，适用于需要探索与规划的复杂推理任务。

🔬 前沿研究（2 项）
[2023] Tree of Thoughts: Deliberate Problem Solving with Large Language Models：首次系统提出 ToT 框架，在 Game of 24、创意写作和迷你填字游戏上超越 CoT。检索关键词：tree of thoughts deliberate problem solving。
[2024] Graph of Thoughts: Solving Elaborate Problems with Large Language Models：将树结构进一步泛化为有向图，允许推理节点合并与复用，降低冗余探索。检索关键词：graph of thoughts LLM reasoning。

🔗 相关概念（3 个）
Chain of Thought：ToT 的父概念；CoT 沿单条线性路径推理，ToT 将其扩展为可分支、可回溯的树搜索。
Self-Consistency：互补策略；Self-Consistency 多次采样后投票，不做路径评估与剪枝；ToT 主动评估并选择最优路径。
MCTS（蒙特卡洛树搜索）：类比概念；ToT 借鉴了 MCTS 的搜索-评估-回溯思想，但用 LLM 自身充当状态生成器与评估器。

⚖️ 优缺点
优点：① 支持回溯与规划，能纠正错误推理方向；② 在需要探索多方案的复杂任务上显著优于线性推理。
缺点：① 树搜索导致 LLM 调用次数大幅增加，成本远高于 CoT；② 需设计任务特定的状态分解与评估策略，通用性受限。