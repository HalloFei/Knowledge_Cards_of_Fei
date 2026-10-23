{
  "term": "Rubric Prompt",
  "full_form": "Rubric Prompt",
  "folder": "Method",
  "definition": "嵌入评分标准的提示词，引导LLM按量规进行结构化评估",
  "tags": ["evaluation", "prompt-engineering", "llm-as-judge"],
  "storage_path": "KB/Method/rubric-prompt.md"
}

📖 详细定义
Rubric Prompt 是一种将评分量规（Rubric）直接嵌入 Prompt 的技术，向 LLM 明确给出各评分等级的具体标准与描述，使模型能像人类评委一样按预定义维度和刻度进行一致、可重复的打分，是 "LLM-as-Judge" 范式的核心实现手段。
🔬 前沿研究
[2023] Judging LLM-as-a-Judge with MT-Bench：引入结构化 rubric 显著提升 LLM 评判一致性，与人类评分相关性达 80%+，关�词：MT-Bench rubric。
[2024] Calibrated Rubrics for Automated Evaluation：研究如何设计校准过的 rubric 减少 LLM 评分偏差（位置偏差、长度偏差），关键词：calibrated rubric LLM evaluation。
🔗 相关概念
LLM-as-Judge：父概念——Rubric Prompt 是实现 LLM-as-Judge 的关键技术，提供评分依据。
Chain-of-Thought (CoT)：互补——在 Rubric Prompt 中结合 CoT 让模型先推理再打分，提升评估质量。
Reference-free Evaluation：关联——Rubric Prompt 使无参考答案的开放式评估成为可能，仅凭量规标准打分。
⚖️ 优缺点
优点：1. 评分标准显式化，可复现且易审计；2. 大幅降低人工评估成本，可规模化部署。
缺点：1. Rubric 设计质量决定评估质量，差的量规导致一致性差；2. LLM 仍存在固有偏差（冗长偏好、位置效应），rubric 无法完全消除。