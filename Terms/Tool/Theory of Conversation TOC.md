{
  "term": "TOC",
  "full_form": "Theory of Conversation (可能歧义：Table of Contents / Theory of Constraints / Token-Optimized Chat)",
  "folder": "Tool",
  "definition": "面向真人化对话体验的 LLM 应用/框架，强调自然感、情感与上下文连续性",
  "tags": ["llm", "conversational-ai", "chatbot", "human-like"],
  "storage_path": "Terms/Tool/Theory of Conversation TOC.md"
}
```

## 📖 详细定义
TOC 在本语境下指一类以"拟人化对话"为核心目标的 LLM 应用/系统，通过人格设定、情绪建模、长期记忆和多轮上下文管理，使对话更接近真人交流；区别于任务型 Chatbot，其关注点是"聊感"而非"完成率"。

## 实际应用
当前该方向是 C 端 AI 陪伴/社交赛道的主流形态，代表产品包括 Character.AI、Replika、Talkie、豆包/星野等；2025-2026 年头部厂商普遍引入长期记忆（Memory）、语音实时对话（Realtime API）与多模态表情反馈。竞争方案有传统任务型 Assistant（ChatGPT/Claude）与角色扮演型 RP-LLM，两者边界正逐步融合。

## 🔬 前沿研究（2 项）
- `[2024] Character-LLM / Role-Play LLM`：通过角色人格数据微调，让模型稳定维持一致人设。检索关键词：`role-play LLM persona consistency`。
- `[2025] Long-term Memory for Conversational Agents`：使用外部向量记忆 + 反思机制实现跨会话记忆。检索关键词：`MemGPT / long-term memory chatbot`。

## 🔗 相关概念（3 个）
- `Chatbot`：父概念，TOC 是其"拟人化"子类。
- `Agent`：互补，Agent 强任务执行，TOC 强情感陪伴。
- `RAG`：易混淆，RAG 解决知识准确性，TOC 解决对话真实感，可组合使用。

## ⚖️ 优缺点
- **优点**：用户粘性高，情感价值突出；对模型幻觉容忍度更高，产品化门槛低于任务型 Agent。
- **缺点**：伦理与依恋风险显著；缺乏客观评测指标，"像真人"难以量化优化。