{
  "term": "Self-Consistency",
  "full_form": "Self-Consistency",
  "folder": "Method",
  "definition": "多次采样推理路径并取多数投票，提升CoT推理鲁棒性的方法",
  "tags": ["prompting", "reasoning", "llm"],
  "storage_path": "KB/Method/self-consistency.md"
}

📖 详细定义
Self-Consistency 是对 Chain-of-Thought 的增强策略：对同一问题多次采样生成不同的推理链与答案，然后以多数投票（majority vote）选择最终答案。其核心假设是——正确推理路径更可能收敛到同一答案，而错误推理路径则分散多样。

🔬 前沿研究（2 项）
[2022] Self-Consistency Improves Chain of Thought Reasoning with Language Models：首次提出该方法，在 GSM8K 等算术推理基准上显著超越单次 CoT。检索关键词：self-consistency chain-of-thought voting。
[2023] Universal Self-Consistency：将投票机制扩展至无标准答案的开放式生成任务，用模型自身判断答案等价性。检索关键词：universal self-consistency open-ended generation。

🔗 相关概念（3 个）
Chain of Thought：Self-Consistency 的父概念与基础；CoT 生成单条推理链，Self-Consistency 对其多次采样再聚合。
Beam Search：易混淆；Beam Search 在解码时保留多条候选序列但基于概率剪枝，Self-Consistency 在答案层面投票而非 token 层面搜索。
Ensemble Learning：类比概念；Self-Consistency 可视为同一模型不同推理路径的"集成"，传统集成则组合多个不同模型。

⚖️ 优缺点
优点：① 无需额外训练或微调即可显著提升推理准确率；② 对错误推理链具有天然容错能力，鲁棒性强。
缺点：① 需多次采样，推理成本线性增长（通常 k=5~40）；② 仅适用于答案空间有限且可比较的任务，开放式生成场景受限。