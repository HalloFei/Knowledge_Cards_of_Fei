{
  "term": "APO",
  "full_form": "Automatic Prompt Optimization",
  "folder": "Method",
  "definition": "借助数值梯度下降类比自动迭代优化提示词的非参数方法",
  "tags": ["prompt-engineering", "llm", "optimization", "beam-search"],
  "aliases": ["Automated Prompt Optimization", "自动提示优化"],
  "storage_path": "Terms/Method/Automatic Prompt Optimization APO.md"
}

## 📖 详细定义
Automatic Prompt Optimization（APO）由 Microsoft Research（Reid Pryzant 等，EMNLP 2023）提出，是一种**非参数**的自动提示优化方法。其核心创新在于将数值梯度下降类比到自然语言空间：用 mini-batch 数据构造自然语言「梯度」——即对当前 prompt 的批评文本，然后沿梯度的**反语义方向**编辑 prompt，配合 beam search 和 bandit selection 引导搜索，自动找到更优的 prompt。

## 实际应用
APO 属于 Automatic Prompt Engineering（APE）技术路线，同路线还有 Google DeepMind 的 OPRO。主要应用于需要高质量 prompt 但人工调优成本高的场景，如分类、推理、代码生成等任务的 prompt 自动优化。目前仍是学术界活跃方向，工业界大规模采用较少，但 PromptAgent、DSPy 等框架已将类似思想工程化。竞争方案包括 OPRO（LLM as Optimizer）、APE（generate-select-resample）等。

## 🔬 前沿研究（2 项）
- `[2023] Automatic Prompt Optimization with "Gradient Descent" and Beam Search (EMNLP 2023)`：APO 原论文，首次提出用自然语言梯度下降 + beam search 自动优化 prompt，在多个 NLU 基准上超越人工设计 prompt（检索关键词：automatic prompt optimization gradient descent beam search）。
- `[2024] AMPO: Automatic Multi-Branched Prompt Optimization (arXiv:2410.08696)`：APO 的后续扩展，从单一流程 prompt 扩展到多分支结构，能处理更复杂的多样化模式任务（检索关键词：AMPO multi-branched prompt optimization）。

## 🔗 相关概念（3 个）
- `OPRO (Optimization by PROmpting)`：直接竞争——Google DeepMind 提出的用 LLM 作为优化器的方法，与 APO 同属自动 prompt 优化路线，区别在于 OPRO 直接用 LLM 生成新候选 prompt 而非梯度编辑。
- `APE (Automatic Prompt Engineer)`：父概念——Zhou 等（2022）提出的自动 prompt 工程框架，采用 candidate → selection → resample 流程，是 APO 等方法的前身。
- `DSPy`：互补——Stanford 提出的编程框架，将 prompt 优化抽象为编译问题，与 APO 的梯度搜索思路互补，DSPy 更侧重模块化与工程化。

## ⚖️ 优缺点
- **优点**：① 非参数、无需微调模型，仅通过黑盒 API 即可优化 prompt；② beam search + bandit 选择提供结构化搜索，比纯随机搜索效率高
- **缺点**：① 需要大量 LLM API 调用，优化成本随搜索空间指数增长；② 对主观任务（如创意写作）效果有限，因缺乏客观评价指标