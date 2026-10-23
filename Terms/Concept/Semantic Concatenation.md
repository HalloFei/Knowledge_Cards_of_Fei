{
  "term": "语义拼接",
  "full_form": "Semantic Concatenation",
  "folder": "Concept",
  "definition": "将不同语义片段按语义关联性融合为完整表示的技术",
  "tags": ["nlp", "embedding", "representation", "fusion"],
  "storage_path": "Terms/Concept/Semantic Concatenation.md"
}

## 📖 详细定义
语义拼接指将多个独立的语义片段（如文本块、向量表示、知识片段）按照语义关联性进行结构化融合，生成一个语义连贯的完整表示。与简单的字符串拼接不同，语义拼接强调拼接后结果的语义完整性与上下文一致性，常涉及向量空间中的特征融合或注意力机制引导的跨片段聚合。

## 实际应用
语义拼接广泛应用于RAG系统中的多段落知识融合、多模态理解中的跨模态特征拼接、以及对话系统中上下文历史与当前查询的融合。在当前LLM应用中，语义拼接是检索增强生成管线的核心环节之一，主流方案包括基于注意力机制的Cross-Attention拼接和基于向量加权融合的拼接策略。竞争方案包括简单拼接（无语义对齐）和端到端联合编码（不显式拼接）。

## 🔬 前沿研究（2 项）
- `[2024] Contextual Semantic Stitching for Long-Document Understanding`：提出基于语义缝合法的长文档理解框架，提升跨段落信息聚合质量。（检索关键词：semantic stitching LLM）
- `[2023] Fusion-in-Decoder Revisited with Semantic Alignment`：改进FiD架构中的语义对齐机制，使多片段拼接的语义一致性显著提升。（检索关键词：fusion-in-decoder semantic alignment）

## 🔗 相关概念（3 个）
- `Feature Concatenation`：父概念。语义拼接是特征拼接的语义增强版本，传统特征拼接仅做维度维度的简单连接。
- `RAG（检索增强生成）`：互补关系。语义拼接是RAG系统中多检索片段融合的关键技术，直接影响生成质量。
- `Cross-Attention`：子概念。Cross-Attention是实现语义拼接的主流机制之一，通过注意力权重实现跨片段的语义关联。

## ⚖️ 优缺点
- **优点**：1) 相比简单拼接能保留更多语义关联信息，减少信息损失；2) 在多源信息融合场景下，拼接结果的上下文一致性更好。
- **缺点**：1) 引入语义对齐/注意力机制后计算复杂度上升；2) 拼接策略对下游任务敏感，需针对场景调优，泛化性有限。