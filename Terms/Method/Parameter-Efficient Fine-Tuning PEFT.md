{
  "term": "PEFT",
  "full_form": "Parameter-Efficient Fine-Tuning",
  "folder": "Method",
  "definition": "仅更新少量参数即可适配大模型到下游任务的微调范式",
  "tags": ["llm", "peft", "fine-tuning", "adaptation"],
  "storage_path": "Terms/Method/parameter-efficient-fine-tuning-peft.md"
}
```

## 📖 详细定义
PEFT 是一类微调方法的统称，其核心思想是冻结预训练大模型的绝大部分参数，仅训练少量新增或选定的参数（通常 <1%），以在显存与算力受限条件下逼近全参数微调（Full Fine-Tuning）的效果。

## 实际应用
截至 2026 年，PEFT 已成为开源社区与企业内部微调 LLM 的**主流范式**。Hugging Face `peft` 库是事实标准，集成 LoRA、QLoRA、Prompt Tuning、Prefix Tuning、IA³、AdaLoRA、DoRA 等方法。在生产中，LoRA/QLoRA 占据绝对主导，其它方法多用于研究或特定场景；全量微调仅在闭源大厂追求极限性能或做持续预训练时使用。

## 🔬 前沿研究（2 项）
- `[2024] DoRA: Weight-Decomposed Low-Rank Adaptation`：权重方向-幅度分解，缩小 PEFT 与全微调的性能差距。
- `[2024] GaLore: Gradient Low-Rank Projection`：不是新增参数，而是将梯度投影到低秩空间，实现全参数训练下的显存节省，模糊 PEFT 与全微调边界。

## 🔗 相关概念（3 个）
- `LoRA`：子概念，PEFT 家族中最主流的具体方法。
- `Full Fine-Tuning`：对立概念，更新全部参数，效果上限更高但成本极大。
- `Prompt Tuning`：兄弟概念，同为 PEFT，通过学习软提示词而非修改模型权重。

## ⚖️ 优缺点
- **优点**：
  - 显存与存储开销极低，多任务只需保存多份小 adapter。
  - 训练快、易于共享分发，适配消费级 GPU 与云端多租户场景。
- **缺点**：
  - 在分布差异大或需要注入大量新知识的任务上，效果仍可能落后全微调。
  - 方法众多、超参敏感，方法/秩/位置选择缺乏统一理论指导。