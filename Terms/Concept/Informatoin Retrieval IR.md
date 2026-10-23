{
  "term": "IR",
  "full_form": "Information Retrieval",
  "folder": "Concept",
  "definition": "从大规模数据中检索相关信息的技术，RAG的核心组件",
  "tags": ["rag", "retrieval", "search"],
  "storage_path": "KB/Concept/information-retrieval.md"
}

📖 详细定义
IR（信息检索）是从大规模文档集合中根据查询找到最相关信息的学科与技术。在 RAG 中，IR 是核心前置环节：先由检索器（Retriever）从知识库中召回相关片段，再送入生成器（Generator）合成答案，检索质量直接决定 RAG 的上限。
🔬 前沿研究
[2023] Dense Retrieval vs Sparse Retrieval in RAG：研究表明混合检索（Dense+Sparse）在 RAG 中优于单一方法，关键词：hybrid retrieval RAG。
[2024] REPLUG & Retrieval Augmentation：将检索结果作为 LLM 上下文注入，显著降低幻觉，关键词：retrieval-augmented LLM。
🔗 相关概念
RAG (Retrieval-Augmented Generation)：父子关系——IR 是 RAG 的"检索"部分，RAG = IR + Generation。
Dense Retrieval：子概念——用向量嵌入进行语义检索，是 RAG 中主流的 IR 方式。
BM25：互补——经典稀疏检索算法，常与 Dense Retrieval 组成混合检索提升 RAG 效果，见 [BM25](../Method/BM25.md)。
⚖️ 优缺点
优点：1. 使 RAG 能访问训练数据之外的知识，无需重训练模型；2. 检索结果可溯源，增强答案可解释性。
缺点：1. 检索噪声或无关文档会误导生成器，产生"检索污染"；2. 稠密检索对嵌入模型质量高度依赖，领域迁移成本高。
已为术语 IR (Information Retrieval) 生成结构化归档卡片与精简学习笔记。IR 是 RAG 的核心检索组件，检索质量直接决定 RAG 的效果上限。