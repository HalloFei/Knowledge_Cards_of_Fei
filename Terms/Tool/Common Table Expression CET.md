{
  "term": "CET",
  "full_form": "Common Table Expression (通常缩写为 CTE，用户写作 CET 可能为笔误)",
  "folder": "Tool",
  "definition": "SQL 中用 WITH 定义的临时命名结果集，供后续查询引用",
  "tags": ["sql", "database", "query"],
  "storage_path": "Terms/Tool/Common Table Expression CET.md"
}

## 📖 详细定义
CTE（Common Table Expression，公用表表达式）通过 `WITH` 子句在单条 SQL 语句作用域内定义一个具名的临时结果集，可在随后的 `SELECT/INSERT/UPDATE/DELETE` 中像表一样引用，用于拆分复杂查询、提高可读性，并支持递归查询。

## 实际应用
CTE 是当前主流关系型数据库（PostgreSQL、MySQL 8.0+、SQL Server、Oracle、SQLite 3.8.3+）的标准特性，广泛用于报表拆解、层级/图结构递归遍历（如组织架构、BOM 展开）、以及替代子查询提升可维护性。竞争方案包括临时表、派生表（子查询）和视图；相比之下 CTE 生命周期仅限当前语句，无需 DDL 权限，是分析型 SQL 的首选写法之一。

## 🔬 前沿研究（2 项）
- `[2023] Recursive CTE optimization in modern DBMS`：探讨递归 CTE 的物化策略与并行执行优化。检索关键词：recursive CTE optimization pipeline。
- `[2024] Materialization hints for CTEs`：PostgreSQL 12+ 引入 `MATERIALIZED/NOT MATERIALIZED` 提示，研究其对查询计划的影响。

## 🔗 相关概念（3 个）
- `Subquery`：父/替代关系——CTE 可视为具名、可复用的子查询，可读性更好。
- `View`：互补——View 是持久化 schema 对象；CTE 是语句级临时定义。
- `Recursive Query`：子——递归 CTE 是 CTE 的特例，用于处理层级/图数据。

## ⚖️ 优缺点
- **优点**：显著提升复杂 SQL 可读性；原生支持递归查询。
- **缺点**：部分数据库中默认物化可能阻碍优化器；作用域仅限单条语句，无法跨查询复用。