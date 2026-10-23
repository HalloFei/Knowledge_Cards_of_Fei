---
term: DAG
full_form: Directed Acyclic Graph
folder: Concept
definition: 有方向且无环路的有向图，用于表达元素间的依赖与偏序关系
tags: [graph, scheduling, data-flow]
storage_path: KB/Concept/dag.md
---

## 📖 详细定义
- DAG 是由顶点和有向边组成的图，其中不存在从某顶点出发沿边方向又能回到该顶点的路径。它天然表达偏序关系，广泛用于任务调度、数据血缘追踪、因果推断和编译器基本块依赖分析。

## 实际应用
- 当前主流：Apache Airflow / Spark / dbt 等数据工程工具均以 DAG 描述任务编排；Git 的提交历史本身就是一个 DAG。
- 在 LLM 领域，多步推理链（如 ToT）和 Agent 工作流编排也用 DAG 建模执行依赖。
- 区块链中 IOTA Tangle 用 DAG 替代链式结构实现并行确认，是活跃的工程方向。

## 🔬 前沿研究（2 项）
- `[2024] DAG-Based Causal Discovery under Latent Confounders`：在隐变量下利用 DAG 结构约束改进因果发现效率（检索关键词：DAG causal discovery latent）。
- `[2023] D3AG: Distributed DAG Scheduling for Large-Scale Workflows`：面向大规模工作流的分布式 DAG 调度框架，降低关键路径延迟（检索关键词：distributed DAG scheduling）。

## 🔗 相关概念（3 个）
- `Tree`：DAG 的特例——若每个节点至多一个父节点，DAG 退化为树；树更严格但表达力更弱。
- `Topological Sort`：DAG 上的线性化算法，将偏序转为全序，是调度与编译的核心步骤。
- `Petri Net`：与 DAG 同为并发建模工具，但 Petri Net 允许回路并显式建模并发与冲突，DAG 仅表达静态依赖。

## ⚖️ 优缺点
- **优点**：1. 天然表达偏序/依赖，无环保证可拓扑排序；2. 支持并行度分析，便于调度优化。
- **缺点**：1. 无法表达循环或反馈依赖，需额外机制处理；2. 大规模 DAG 的可达性查询代价较高。