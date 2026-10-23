```json
{
  "term": "PDDL",
  "full_form": "Planning Domain Definition Language",
  "folder": "Tool",
  "definition": "自动规划领域与问题的标准化建模语言，STRIPS 语义的现代继承者",
  "tags": ["planning", "ai", "strips", "formal-language", "agent"],
  "storage_path": "Terms/Tool/Planning Domain Definition Language PDDL.md"
}
```

## 📖 详细定义

- PDDL 是国际规划竞赛（IPC）自 1998 年起的官方输入语言，把一个规划任务拆成两个文件：**domain**（定义类型、谓词、动作 schema——每个动作含 `:precondition` / `:effect`，其中 effect 由 add 与 delete 组合而成，直接继承 STRIPS 语义）和 **problem**（定义具体的对象、初始状态、目标）。语言分级递进（PDDL 1.2 = STRIPS+ADL、2.1 加数值与时序动作、2.2 加派生谓词、3.0 加偏好与硬约束、3.1 加对象流），让求解器可按能力订阅子集。

## 实际应用

- 学术界规划竞赛与 benchmark 的事实标准，Fast Downward、LAMA、ENHSP 等主流求解器都以 PDDL 为输入；工业界在物流调度、航天任务规划（NASA MAPGEN）、机器人任务分配里仍在用。**近两年真正的热度来自 LLM 结合**：把自然语言目标翻译成 PDDL、交给经典求解器出可验证计划（LLM+P、Planetarium、TIC），比让 LLM 直接生成动作序列可靠得多。这条路线目前是 neuro-symbolic agent 的主流之一，但仍限于问题结构清晰、谓词封闭的领域；开放世界（web、GUI）里 PDDL 的建模成本本身就是瓶颈。

## 🔬 前沿研究（2 项）

- `[2023] LLM+P: Empowering Large Language Models with Optimal Planning Proficiency`：LLM 把自然语言问题翻成 PDDL，交经典 planner 求解，再翻译回自然语言，比纯 LLM 规划正确率高一个数量级（关键词：LLM+P Liu 2023）。
- `[2024] Planetarium: A Rigorous Benchmark for Translating Text to Structured Planning Languages`：系统评测 LLM 生成 PDDL 的正确性，暴露"看起来像 PDDL 但语义错误"这类失败模式（关键词：Planetarium PDDL benchmark）。

## 🔗 相关概念（3 个）

- `STRIPS`：父关系——PDDL 的最小子集就是 STRIPS 的 add/delete 语义，其余层级都是在这个骨架上加表达力（类型、量化、时序、数值）。
- `HTN（Hierarchical Task Network）`：互补关系——HTN 用"任务分解"规划，PDDL 用"状态转移"规划；HDDL（2020）把 HTN 融进 PDDL 家族，二者不再互斥。
- `ADL（Action Description Language）`：兄弟关系（易混淆）——ADL 是 1987 年 Pednault 在 STRIPS 之上加的条件效果与量化，PDDL 1.x 主要就是"STRIPS + ADL"的合并规范化。

## ⚖️ 优缺点

- **优点**：1. 标准化+分级，同一份 domain 可以喂给几十种求解器，生态成熟；2. 语义严格，规划结果可形式验证，天然适合"LLM 出草稿、planner 兜底"的 neuro-symbolic 组合。
- **缺点**：1. 手写 domain 门槛高、维护成本大，工业界很难覆盖真实业务的全部前提与效果（qualification problem）；2. 高阶特性（连续时序、概率、部分可观测）求解器支持稀疏，一旦离开经典离散确定假设就没什么现成工具可用。