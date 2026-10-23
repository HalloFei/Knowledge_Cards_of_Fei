---
term: SCM
full_form: Structural Causal Model
folder: Concept
definition: 用结构方程和DAG描述变量间因果机制的数学框架
tags: [causality, dag, statistics, machine-learning]
storage_path: KB/Concept/scm.md
---

## 📖 详细定义
- SCM 由 Judea Pearl 提出，包含三要素：一组内生/外生变量、一组结构方程（描述每个变量如何由其直接原因决定）、以及一个 DAG（编码因果方向）。SCM 支持三层因果推理——关联（观测）、干预（do-calculus）和反事实，是因果推断的统一理论基础。

## 实际应用
- 流行病学与医学：用 SCM 识别混杂因素，估计治疗效果（如疫苗有效性评估）。
- 推荐系统/广告归因：通过 SCM 建模用户行为因果链，去除混杂偏差，提升归因准确。
- LLM 可解释性：近期研究用 SCM 描述模型内部因果机制（如 causal tracing），分析知识存储位置。

## 🔬 前沿研究（2 项）
- `[2024] Causal Abstraction for LLM Interpretability`：用 SCM 作为抽象层解释 Transformer 内部计算的因果结构（检索关键词：causal abstraction LLM interpretability）。
- `[2023] Causal Representation Learning`：将 SCM 与表示学习结合，从观测数据中自动发现因果变量和结构方程（检索关键词：causal representation learning SCM identifiability）。

## 🔗 相关概念（3 个）
- `DAG`：子组件——SCM 中用 DAG 编码变量间因果方向，DAG 是 SCM 的图形化表示。
- `Do-Calculus`：子方法——Pearl 提出的干预推理规则集，基于 SCM 的图结构判断因果效应可识别性。
- `Bayesian Network`：易混淆——叶斯网络也用 DAG 但仅编码条件独立性（关联层），SCM 额外支持干预和反事实。

## ⚖️ 优缺点
- **优点**：1. 统一关联/干预/反事实三层推理，理论表达力最强；2. 图形化直观，便于领域专家编码先验知识。
- **缺点**：1. 需要先验因果图，大规模变量下结构学习困难；2. 结构方程的参数化假设可能与真实数据不符。