```json
{
  "term": "GOAP",
  "full_form": "Goal-Oriented Action Planning",
  "folder": "Method",
  "definition": "以目标为导向、结合 A* 搜索与算子代价的实时规划方法",
  "tags": ["planning", "ai", "game-ai", "a-star"],
  "storage_path": "KB/Method/goap.md"
}
```

## 📖 详细定义

- GOAP 源自 STRIPS 的状态-算子模型，但引入了算子代价（cost）和 A* 启发式搜索，使智能体能根据当前世界状态和目标自动规划行动序列。与传统固定行为树不同，GOAP 在运行时动态生成计划，广泛应用于游戏 NPC 行为系统。

## 🔬 前沿研究（2 项）

- `[2023] Neural GOAP for Game NPCs`：用神经网络学习算子代价函数，替代手工调参，使 GOAP 在动态游戏环境中自适应调整行为（关键词：neural GOAP cost learning）。
- `[2024] Multi-Agent GOAP`：扩展 GOAP 至多智能体协作规划，引入共享世界状态与冲突消解机制（关键词：multi-agent GOAP）。

## 🔗 相关概念（3 个）

- `STRIPS planning`：父关系——GOAP 继承 STRIPS 的状态-算子表示（前提/添加/删除），并在此基础上引入代价搜索。
- `HTN（分层任务网络）`：互补关系——HTN 依靠预定义的任务分解结构，GOAP 依靠运行时搜索，前者更适合结构化场景，后者更适合开放动态场景。
- `Behavior Tree（行为树）`：互补关系——行为树是静态优先级驱动的决策结构，GOAP 是动态规划驱动的决策结构，两者可混合使用。

## ⚖️ 优缺点

- **优点**：1. 运行时动态规划，行为灵活且无需预编写固定流程；2. A* 代价搜索使计划具有最优性倾向，便于权衡行为优先级。
- **缺点**：1. 每次规划需执行搜索，计算开销高于行为树等静态方案；2. 算子代价和启发式函数需手工调优，不当设定会导致次优或无效计划。