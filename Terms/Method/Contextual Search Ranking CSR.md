{
  "term": "CSR",
  "full_form": "Contextual Search Ranking",
  "folder": "Method",
  "definition": "结合用户上下文信息的搜索结果排序方法",
  "tags": ["search", "ranking", "context", "information-retrieval"],
  "storage_path": "Terms/Method/Contextual Search Ranking CSR.md"
}

## 📖 详细定义
Contextual Search Ranking（上下文搜索排序）指在信息检索系统中，除查询词本身外，还综合用户历史行为、地理位置、时间、设备、会话上下文等信号对候选文档进行个性化排序的方法。核心在于将静态的 query-document 相关性扩展为动态的 (query, user, context)-document 相关性。

## 实际应用
CSR 是现代搜索引擎（Google、Bing、百度）与推荐系统的核心模块之一，常见于电商搜索、个性化新闻流、企业知识库检索等场景。主流方案包括 Learning-to-Rank (LTR) + 特征工程、深度向量检索 + 重排序，以及基于 LLM 的语义匹配。竞争/互补技术包括传统 [BM25](BM25.md) 排序、Collaborative Filtering、Learning-to-Rank (LTR)。

## 🔬 前沿研究（2 项）
- `[2023] Transformer-based Contextual Ranking`：将用户历史序列与 query 联合编码，实现端到端上下文感知排序。检索关键词：transformer contextual ranking user sequence。
- `[2024] LLM-enhanced Search Ranking`：利用大语言模型生成查询意图扩展与上下文理解，提升长尾查询排序效果。

## 🔗 相关概念（3 个）
- `Learning-to-Rank (LTR)`：父——CSR 常通过 LTR 模型实现，是具体应用方向。
- `Personalized Search`：子——个性化搜索是 CSR 的典型子任务。
- `Query Intent Understanding`：互补——意图识别为 CSR 提供语义层面上下文信号。

## ⚖️ 优缺点
- **优点**：显著提升搜索相关性与用户体验；支持个性化与场景化排序。
- **缺点**：对用户隐私数据依赖强，存在合规风险；冷启动场景下效果受限。