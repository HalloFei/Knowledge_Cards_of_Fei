```json
{
  "term": "HTN",
  "full_form": "Hierarchical Task Network",
  "folder": "Method",
  "definition": "通过递归分解高层任务为可执行子任务的层次化规划方法",
  "tags": ["planning", "ai", "decomposition", "hierarchy"],
  "storage_path": "KB/Method/htn.md"
}
```

## 📖 详细定义

- HTN 将规划问题建模为任务分解过程：高层抽象任务通过方法（method）递归分解为子任务网络，直至所有子任务为可直接执行的原语操作（primitive action）。与状态空间规划不同，HTN 的搜索驱动力来自任务结构而非目标状态。

## 🔬 前进研究（2 项）

- `[2023] LLM-Guided HTN Decomposition`：利用大语言模型生成 HTN 任务分解策略，减少传统 HTN 对专家编写的分解规则的依赖（关键词：LLM+HTN decomposition）。
- `[2024] HTN with Learned Heuristics`：通过强化学习训练 HTN 求解器的分解启发式，在 IPC 基准上显著提升搜索效率（关键词：learned HTN heuristics）。

## 🔗 相关概念（3 个）

- `STRIPS planning`：互补关系——STRIPS 按状态变化驱动规划，HTN 按任务分解驱动规划，两者可在混合框架中结合。
- `PDDL`：互补关系——PDDL 2.2 引入 HTN 扩展，使 HTN 方法可在 PDDL 框架内表达与求解。
- `GOAP（Goal-Oriented Action Planning）`：互补关系——GOAP 采用状态驱动搜索但引入代价启发式，HTN 采用任务分解但可嵌入状态约束，两者在游戏 AI 中各有适用场景。

## ⚖️ 优缺点

- **优点**：1. 利用领域知识的任务结构，大幅缩减搜索空间；2. 分解过程天然产生可解释的层次化计划。
- **缺点**：1. 依赖专家手工编写分解方法，领域建模成本高；2. 缺乏目标驱动的灵活性，当任务分解路径不可行时回溯代价大。