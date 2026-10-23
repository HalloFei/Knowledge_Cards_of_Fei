{
  "term": "RASA",
  "full_form": "RASA Open Source (可能歧义：Rasa NLU/Rasa Core 为其子组件)",
  "folder": "Tool",
  "definition": "开源对话式AI框架，用于构建上下文感知的聊天机器人",
  "tags": ["chatbot", "nlu", "open-source"],
  "storage_path": "KB/Tool/rasa.md"
}

📖 详细定义
RASA 是一个开源的对话式 AI 框架，由 Rasa NLU（自然语言理解）和 Rasa Core（对话管理）两大模块组成，支持意图识别、实体抽取与基于机器学习的对话策略，使开发者能在本地部署并完全控制数据与模型。
🔬 前沿研究
[2023] Rasa Pro & CALM：Rasa 推出 CALM（Conversational AI with Language Models）架构，将大语言模型融入对话管理，关键词：Rasa CALM。
[2024] LLM-augmented Rasa Pipelines：探索将 LLM 作为 NLU 后备提升意图识别鲁棒性，关键词：LLM Rasa integration。
🔗 相关概念
Dialogflow：互补/竞争——Google 的云端对话平台，Rasa 为开源本地化替代方案。
NLU (Natural Language Understanding)：父概念——Rasa NLU 实现了 NLU 的核心功能（意图+实体）。
CDD (Conversation Driven Development)：互补——Rasa 官方推荐 CDD 作为其开发方法论，对话数据驱动模型迭代。
⚖️ 优缺点
优点：1. 完全开源可本地部署，数据隐私可控；2. 支持自定义 ML 管道与对话策略，灵活度高。
缺点：1. 学习曲线较陡，配置与训练流程复杂；2. 对大规模预训练语言模型的原生支持不如新兴框架。