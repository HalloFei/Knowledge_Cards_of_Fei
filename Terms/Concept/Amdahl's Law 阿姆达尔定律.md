```json
{
  "term": "Amdahl's Law",
  "full_form": "Amdahl's Law（阿姆达尔定律）",
  "folder": "Concept",
  "definition": "固定问题规模下，系统加速比的上限由不可并行部分的时间占比决定：S(n)=1/(s+p/n)",
  "tags": ["performance", "parallelism", "scaling", "system"],
  "aliases": ["阿姆达尔定律", "Amdahl 定律", "强扩展上限"],
  "storage_path": "Terms/Concept/Amdahl's Law 阿姆达尔定律.md"
}
```

## 📖 详细定义

Gene Amdahl 1967 年在 AFIPS 上给出的上界：把程序耗时拆成不可并行部分 s（serial fraction）与可并行部分 p = 1 − s，用 n 个处理器加速后

`S(n) = 1 / (s + p/n) = 1 / ((1 − p) + p/n)`

n → ∞ 时 S_max = 1/s。s = 2% 时天花板是 50×，第 100 个核相比第 50 个核的边际收益已不足 1%。并行效率 E(n) = S(n)/n 同步衰减。

三个容易被忽略的前提：

1. **固定问题规模**（强扩展 strong scaling）。问题随机器一起放大是另一条曲线——Gustafson 的弱扩展 S(n) = s + p·n，同样时间里 n 核能算 n 倍工作量。
2. **p 是时间份额，不是 FLOP 份额**。把「GEMM 占 90% FLOPs」直接代进 p 是最常见的错误；必须用 wall-clock profile。内存受限的 decode 阶段 FLOPs 占比很低，却占据大半时间。
3. **s 是当前实现的设计约束，不是物理常数**。通信、同步、kernel launch、负载不均、串行后处理都计入 s，且其中多数可以被消除（overlap、通信计算融合、异步）。Amdahl 给的是「这一版实现」的天花板。

## 实际应用

- **先测 s 再决定投什么**：性能优化最常见的浪费是把预算投在 p 部分。s = 50% 时，把并行部分做到无穷快也只到 2×，此时正确的动作是攻击 s，而不是加机器。
- **LLM 推理的适用边界**：prefill 是 GEMM 密集型，p 高，Amdahl 近似成立；decode 是带宽受限，加算力不加带宽几乎没有收益，这类问题该用 Roofline 而非 Amdahl。量化（FP8/INT4）把 GEMM 算快了却拿不到端到端等比加速，本质是 decode 那段时间不在「被量化的算力」上。
- **分布式训练**：TP 的 all-reduce、PP 的 bubble、DP 的梯度同步都进入 s；且 s 随节点数上升（通信与同步变贵），所以真实曲线常比 Amdahl 更早压平。
- **评审用法**：当「直觉检查」而非「证据」——可以用它论证方向（这块占比不够大，不值得优化），不要用它论证具体数字（因此应该有 1.8×），除非 s 被真正 profile 过。

## 🔬 前沿研究（3 项）

- `[1967] Amdahl, Validity of the single processor approach to achieving large scale computing capabilities`（AFIPS）：原文，给出固定负载下的加速比上界（检索关键词：Amdahl 1967 single processor approach）。
- `[1988] Gustafson, Reevaluating Amdahl's Law`（CACM）：提出固定时间下规模随核数增长的弱扩展曲线，是集群「线性加速」宣传的理论依据（检索关键词：Gustafson scaled speedup）。
- `[2008] Hill & Marty, Amdahl's Law in the Multicore Era`（IEEE Computer）：把模型推广到对称/非对称多核，给出异构执行下的加速比族（检索关键词：Amdahl multicore era Hill Marty）。

## 🔗 相关概念（3 个）

- `Gustafson's Law`：对偶命题——弱扩展下问题规模可变，加速比接近线性；与 Amdahl 的分歧只在「规模是否固定」。
- `Roofline Model`：互补维度——Amdahl 管并行度，Roofline 管算力 vs 带宽；decode、GEMV 阶段后者解释力更强。
- `Critical Path`：更细粒度的下界——并行任务的耗时由关键路径决定，Amdahl 是「单条串行链 + 均质并行块」假设下的特例。

## ⚖️ 优缺点

- **优点**：① 一个除法就能判断优化方向值不值得投入，是性能工程的第一道筛子；② 把「加机器」和「改代码」的收益放在同一把尺子上；③ 形式简单，适合当评审里的共同语言。
- **缺点**：① s 依赖测量粒度，p 随 batch / shape / 并发度漂移，「一个 s」往往不存在；② 忽略开销、忽略扩展带来的额外通信、假设无限并行，因此是上界不是预测；③ 常被误读成「不要并行」的论据，而 s 恰恰是可被工程改造的对象；④ 对带宽受限（decode、GEMV）与不规则负载（MoE 路由不均）不适用。
