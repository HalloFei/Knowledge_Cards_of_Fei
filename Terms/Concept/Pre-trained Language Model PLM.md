{
  "term": "PLM",
  "full_form": "Pre-trained Language Model",
  "folder": "Concept",
  "definition": "在大规模语料上预训练的语言模型，可微调适配下游任务",
  "tags": ["nlp", "pre-training", "transformer"],
  "storage_path": "KB/Concept/plm.md"
}

📖 详细定义
PLM 指通过自监督目标（如 MLM、CLM）在大规模无标注文本上预训练的语言模型。在 "PLM+LLM 混合架构" 语境中，PLM 通常指参数量较小、可微调的编码器模型（如 BERT、RoBERTa），与超大规模只做推理的 LLM 配合使用——PLM 负责特定领域的精细表示与分类，LLM 负责生成与推理。

🔬 前沿研究（2 项）
[2019] BERT: Pre-training of Deep Bidirectional Transformers：奠定了双向编码器 PLM 范式，引领 "预训练+微调" 流程。检索关键词：BERT pre-training fine-tuning。
[2024] PLM-LLM Hybrid Pipelines for Enterprise NLP：探索用轻量 PLM检索/分类前置、LLM 做生成后置的混合架构，兼顾效率与能力。检索关键词：PLM LLM hybrid pipeline enterprise。

🔗 相关概念（3 个）
LLM：互补概念；LLM 是超大规模自回归模型侧重生成，PLM 通常更小且双向编码侧重理解，混合架构取两者之长。
Fine-tuning：PLM 的核心使用方式；通过在标注数据上微调使通用 PLM 适配特定任务。
Foundation Model：父概念；PLM 和 LLM 都属于基础模型，区别在于规模、架构和使用方式。

⚖️ 优缺点
优点：① 微调成本低，小数据即可获得良好的任务特定表现；② 推理速度快，适合延迟敏感的在线场景。
缺点：① 生成能力弱，难以处理开放式文本生成任务；② 每个任务需单独微调和部署，扩展性不如 LLM 的 in-context learning。