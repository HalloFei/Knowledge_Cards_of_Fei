{
  "term": "LRU",
  "full_form": "Least Recently Used",
  "folder": "Method",
  "definition": "淘汰最久未被访问条目的缓存替换策略",
  "tags": ["cache", "algorithm", "eviction", "offline"],
  "storage_path": "Terms/Method/Least Recently Used LRU.md"
}

## 📖 详细定义
LRU（最近最少使用）是一种缓存淘汰算法：当存容量满时，优先驱逐「距上次访问时间最久」的条目。经典实现为「哈希表 + 双向链表」，读写均为 O(1)。核心假设：近期访问过的数据更可能被再次访问（时间局部性）。

## 实际应用
LRU 至今仍是缓存领域最主流的基线策略，广泛用于 Redis（`allkeys-lru` / `volatile-lru`）、操作系统页面置换、CPU Cache、浏览器/移动端本地缓存等场景。竞争方案包括 LFU（频次优先，抗突发热点）、ARC（自适应）、TinyLFU/W-TinyLFU（Caffeine 默认，命中率优于 LRU）。**在断网降级场景下，纯 LRU 易被一次性浏览行为「污染」，导致高价值内容被驱逐**，因此工业界常采用 LRU + 业务权重的分段/混合策略。

## 🔬 前沿研究（2 项）
- `[2023] LRB (Learning Relaxed Belady)`：用机器学习逼近理论最优 Belady 算法，命中率显著超越 LRU（关键词：learned cache eviction）。
- `[2024] LLM-Cache / Semantic Cache`：面向 LLM 请求的语义相似度缓存，结合 LRU 做二级淘汰（关键词：semantic cache, GPTCache）。

## 🔗 相关概念（3 个）
- `LFU (Least Frequently Used)`：按访问频次淘汰，与 LRU 互补；抗突发扫描但对时间敏感度差（易混淆：LRU 看「时间」，LFU 看「次数」）。
- `TTL (Time To Live)`：基于时间的强制过期，可与 LRU 叠加使用（互补关系）。
- `Belady's Algorithm`：理论最优离线替换算法，LRU 的性能上界参考（父/理论关系）。

## ⚖️ 优缺点
- **优点**：
  1. 实现简单，O(1) 读写，工程成熟度高。
  2. 对具备时间局部性的工作负载命中率良好。
- **缺点**：
  1. 易被一次性扫描/大批量遍历「污染」，热点数据被误驱逐。
  2. 不感知业务价值（如收藏、置顶），需与业务权重结合。

---

## 🎯 针对「断网降级缓存策略」的协同建议
**风险：LRU + 最近展示优先 + 收藏优先如何协同？** 三者若并列会产生优先级冲突。推荐采用**分段缓存 + 加权评分**：

1. **分段（Segmented Cache）**：将缓存空间划分为三区，硬隔离配额，避免互相驱逐。
   - `Pinned 区（收藏/置顶）`：占 30%，**不参与 LRU 淘汰**，仅在用户取消收藏后释放。
   - `Hot 区（最近展示）`：占 40%，存放最近 N 次展示内容，采用 LRU。
   - `Cold 区（其他）`：占 30%，普通 LRU 或 TinyLFU。
2. **统一评分（可选替代方案）**：`score = w1·收藏(0/1) + w2·最近展示时间衰减 + w3·访问频次`，按最低分驱逐；权重建议 `w1=100, w2=10, w3=1`，让收藏近似「不可驱逐」。
3. **降级触发**：断网检测后切换至「只读 Pinned + Hot」模式，Cold 区停止写入，避免离线期间被无效数据填满。
4. **观测指标**：离线命中率、收藏内容缺失率、Cold 区污染率，用于调参分段比例。