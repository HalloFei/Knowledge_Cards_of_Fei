{
  "term": "DST",
  "full_form": "Dialogue State Tracking",
  "folder": "Concept",
  "definition": "在多轮对话中追踪用户意图和槽值的任务模块",
  "tags": ["dialogue", "nlp", "task-oriented"],
  "storage_path": "KB/Concept/dialogue-state-tracking.md"
}

📖 详细定义
DST 是任务型对话系统的核心件，负责在每一轮对话后维护一个结构化的对话状态（通常为 domain-slot-value 三元组集合），反映用户当前的完整需求。它需要处理指代消除、隐式确认、状态继承与修正等现象，为下游的对话策略和回复生成提供决策依据。

🔬 前沿研究（2 项）
[2020] TripPy: A Triple Copy Strategy for Value Independent DST：提出基于 span 抽取与三重拷贝机制的 DST 方法，摆脱对预定义值列表的依赖。检索关键词：TripPy triple copy dialogue state tracking。
[2023] ChatGPT for Zero-Shot Dialogue State Tracking：探索用 LLM 在零样本设定下直接做 DST，通过 prompt 设计实现无需训练的状态追踪。检索关键词：ChatGPT zero-shot DST prompt。

🔗 相关概念（3 个）
NLU（自然语言理解）：DST 的上游模块；NLU 识别单轮意图和槽位，DST 在此基础上跨轮次聚合完整状态。
Dialogue Policy：DST 的下游模块；策略模块基于 DST 输出的状态决定系统下一步动作（查询数据库、追问、确认等）。
Slot Filling：子任务；Slot Filling 关注单轮内识别槽值，DST 需要跨轮维护、更新和纠正这些槽值。

⚖️ 优缺点
优点：① 提供结构化状态表示，使对话系统可解释且便于与后端系统交互；② 解耦对话管理各模块，利于独立优化。
缺点：① 依赖预定义的本体（ontology），新领域迁移需重新定义 schema；② 多领域多槽位场景下状态空间爆炸，标注成本高。