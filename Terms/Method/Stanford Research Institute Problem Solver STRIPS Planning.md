```json
{
  "term": "STRIPS planning",
  "full_form": "Stanford Research Institute Problem Solver Planning",
  "folder": "Method",
  "definition": "基于状态与算子进行形式化表示的自动规划方法",
  "tags": ["planning", "ai", "pddl", "search"],
  "storage_path": "KB/Method/strips-planning.md"
}
```

## 📖 详细定义

- STRIPS 是一种经典自动规划框架，将问题建模为初始状态、目标状态和一组操作算子（action operators），每个算子含前提条件（precondition）、添加列表（add list）和删除列表（delete list）。通过状态空间搜索，从初始状态出发逐步应用算子直至满足目标。

## 🔬 前沿研究（2 项）

- `[2023] Plan-and-Solve Prompting`：将 STRIPS 式规划思想引入 LLM 提示，先规划再执行，提升复杂推理准确率（关键词：plan-and-solve prompting）。
- `[2024] LLM as Planner`：研究用大语言模型替代传统 STRIPS 算子进行开放域规划，结合神经符号混合方法（关键词：neuro-symbolic planning）。

## 🔗 相关概念（3 个）

- `PDDL`：STRIPS 的后继标准语言，子关系——PDDL 语法包含并扩展了 STRIPS 算子表示。
- `HTN（分层任务网络）`：互补关系——HTN 按任务分解规划，STRIPS 按状态变化规划，两者可结合使用。
- `GOAP（Goal-Oriented Action Planning）`：子关系——GOAP 受 STRIPS 启发，将算子代价引入 A* 搜索，广泛用于游戏 AI。

## ⚖️ 优缺点

- **优点**：1. 表示简洁直观，状态-算子模型易于理解和实现；2. 可与多种搜索算法结合，理论成熟。
- **缺点**：1. 仅支持命题级表示，无法表达量化约束和复杂条件；2. 状态空间爆炸问题，大规模问题下搜索效率低。