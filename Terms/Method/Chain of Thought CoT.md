{
  "term": "CoT",
  "full_form": "Chain of Thought",
  "folder": "Method",
  "definition": "让模型逐步推理的提示方法，将复杂问题分解为中间步骤",
  "tags": ["prompting", "reasoning", "llm"],
  "storage_path": "KB/Method/chain-of-thought.md"
}

📖 详细定义
Chain of Thought 是一种提示技术，通过在输入中展示推理步骤的示例（few-shot）或直接指示"逐步思考"（zero-shot），引导大语言模型生成中间推理链后再给出最终答案，显著提升多步推理任务的准确率。

🔬 前沿研究（2 项）
[2022] Chain-of-Thought Prompting Elicits Reasoning in Large Language Models：首次系统提出 CoT 概念，证明其在算术、常识和符号推理上的效果。检索关键词：chain-of-thought prompting reasoning。
[2023] Tree of Thoughts：将线性推理链扩展为树状搜索结构，结合回溯与规划提升复杂问题求解能力。检索关键词：tree of thoughts deliberate search。

🔗 相关概念（3 个）
Few-shot Prompting：CoT 常以 few-shot 形式呈现推理示例，是 CoT 的载体；普通 few-shot 只给答案示例，CoT 额外提供推理过程。
Self-Consistency：CoT 的增强版本，通过多次采样推理路径并取多数投票提升鲁棒性，父概念为 CoT。
ReAct：互补方法；CoT 侧重纯语言推理链，ReAct 将推理与外部工具调用交替进行，弥补知识盲区。

⚖️ 优缺点
优点：① 显著提升多步推理任务表现，无需修改模型参数；② 推理过程可解释，便于调试与错误定位。
缺点：① 推理链越长生成延迟和 token 开销越大；② 模型可能生成看似合理但逻辑错误的推理链（幻觉推理）。