# 路由机制让注意力只看该看的，把二次方变成一点五次方

**Efficient Content-Based Sparse Attention with Routing Transformers**

- date: 2026-07-28
- tags: [paper]
- identifier: 20260728T120000
- source: https://arxiv.org/abs/2003.05997
- authors: Aurko Roy, Mohammad Saffar, Ashish Vaswani, David Grangier
- venue: TACL 2020

## 速读

自注意力让序列中每个位置都能直接关注任意历史位置,但代价是 O(n²d) 的计算和内存——序列一长就撑不住。先前的稀疏注意力要么只看固定窗口(位置稀疏),要么虽然按内容选择却要先算完整注意力矩阵再剪枝(内容稀疏),两头都不干净。Routing Transformer 看见了一件事:如果先把 query 和 key 投到单位球上做 online k-means 聚类,让同一 cluster 里的 token 互相注意,那聚类的分组本身就是内容驱动的稀疏模式,而且不需要先算完整矩阵——聚类中心是模型参数,在线更新,分组结果直接决定注意力模式。这样每个 query 只看 √n 个 key,总复杂度从 O(n²d) 降到 O(n^1.5 d)。在 Wikitext-103 上 perplexity 从 18.3 降到 15.8,PG-19 上从 33.6 降到 33.2。以后遇到长序列建模,先问:注意力的稀疏模式能不能由内容本身决定,而不是靠固定窗口或先算再剪?

## 它到底在解决什么

自回归语言模型处理长文档时,每个位置需要关注之前所有位置来预测下一个词。自注意力让每个位置直接连接所有历史位置,但 n 个位置两两配对产生 n×n 的注意力矩阵——序列长度翻倍,计算量翻四倍。8192 个 token 的序列意味着 6700 万对注意力权重,这在内存和算力上都不可行。

人们先想到的出路是位置稀疏注意力:只看最近 k 个位置(local attention),或者每隔几步看一次(strided attention)。这些方法有效,因为语言中相邻词的关联确实最强。但它们有一个硬伤:稀疏模式在训练前就固定了,不看内容。如果第 100 个词和第 5000 个词语义高度相关(比如同一个专有名词再次出现),local attention 硬是看不到。

另一条路是内容稀疏注意力:用 sparsemax 或 entmax 让注意力权重本身变稀疏。但问题在于,这些方法必须先算出完整的 n×n 注意力矩阵,再从中挑出非零项——计算量没有省,只是推理时可以跳过零项。先花 O(n²d) 再剪枝,不是真正的稀疏。

两条路都没能同时做到两件事:让稀疏模式由内容决定,又不先算完整矩阵。Routing Transformer 就是被这个缺口逼出来的。

## 它真正看见了什么

**核心洞察**:在点积注意力中,query Q_i 对 key K_j 的注意力权重正比于 Q_i^T K_j。这是一个最大内积搜索(MIPS)问题——给定 Q_i,找内积最大的 K_j。而 MIPS 在向量归一化后等价于最近邻搜索(NNS),NNS 又可以用 k-means 聚类来近似。

这个等价关系是整篇论文的脊柱:

```text
  Q_i, K_j 在单位球上
  ||Q_i - K_j||^2 = 2 - 2(Q_i^T K_j)
  距离小  <=>  内积大  <=>  注意力权重高
  => 同一 cluster 的 key 就是高注意力 key 的近似
```

**路由注意力的具体运作**:

1. 将 query 和 key 用 LayerNorm(去掉 scale 和 bias)投影到单位球
2. 用 online k-means 把 n 个 token 分成 k 个 cluster,聚类中心 μ 是模型参数,随训练在线更新(指数移动平均,λ=0.999)
3. 每个 query 只关注被分到同一 cluster 的 key(causal 场景下 Q=K 共享,避免未来信息泄漏)
4. 为了硬件效率,每个 cluster 固定取 top-n/k 个最近 token,保证 cluster 大小均衡

```text
  全注意力矩阵 (n*n):          路由注意力矩阵 (k*(n/k)*(n/k)):
  +--------------------+       +-----+
  |********************|       |*    |  cluster 1: sqrt(n) Q x sqrt(n) K
  |********************|  -->  | *   |  cluster 2: sqrt(n) Q x sqrt(n) K
  |********************|       |  *  |  cluster 3: sqrt(n) Q x sqrt(n) K
  |********************|       |   * |  ...
  +--------------------+       +-----+  k=sqrt(n) clusters
  O(n^2 d)                     O(n^1.5 d)
```

**复杂度为什么是 O(n^1.5 d)**:

- 聚类分配:n 个 token 与 k=√n 个中心做内积 -> O(n√n d) = O(n^1.5 d)
- cluster 内注意力:k 个 cluster,每个 √n 个 query 对 √n 个 key -> k × (√n)² × d = O(n^1.5 d)
- 两项同级,总复杂度 O(n^1.5 d)

**与 Reformer 的关键区别**:Reformer 也做内容稀疏,但用 LSH(随机超平面哈希)来分组,哈希函数是固定的、不随训练更新。Routing Transformer 用可学习的聚类中心,spherical k-means 在 MIPS 任务上已知优于 LSH。ImageNet-64 上 Reformer 3.65 bits/dim vs Routing Transformer 3.43 bits/dim,差距显著。

**与 local attention 的互补性**:论文发现只用路由注意力(不加 local)效果反而差(CIFAR-10 上 3.291 vs 混合的 2.971 bits/dim)。Jensen-Shannon 散度分析显示,routing head 的注意力分布与 local head 几乎正交(JSD 接近上限 0.6931),说明两者捕获了不同类型的依赖:local 保证局部流畅性,routing 保证全局一致性。

**最小解释模型**:

```text
  输入:长序列 token -> 投影到 Q, K, V
       |
  LayerNorm 归一化到单位球
       |
  online k-means 聚类(可学习中心,在线更新)
       |
  每个 query 只看同 cluster 的 sqrt(n) 个 key
       |
  首个可观察变化:注意力矩阵从 n*n 变成 k 个 sqrt(n)*sqrt(n) 块
       |
  结果:O(n^1.5 d) 复杂度 + 内容驱动的稀疏模式
```

**邻近情况区分**:Sparse Transformer(Child et al. 2019)也做稀疏注意力,但它的稀疏模式是固定的(local + strided),不看内容。如果序列中两个语义相关的 token 恰好不在同一个固定窗口或步幅位置,Sparse Transformer 永远无法让它们交互。Routing Transformer 的聚类结果随内容变化,同一个 token 在不同上下文中可能被分到不同 cluster——稀疏模式是数据依赖的。

**条件变化预测**:如果聚类中心不随训练更新(退化为固定哈希),routing attention 就变成一种随机分组,效果应接近 Random Transformer——论文的消融实验证实了这一点:CIFAR-10 上 3.076 vs local 的 3.009 vs routing 的 2.971 bits/dim,随机分组反而比纯 local 更差。这说明可学习的聚类中心是 routing 起作用的关键条件。

**关键证据**:
- Wikitext-103:10 层 Routing Transformer 达到 15.8 perplexity,而 18 层 Transformer-XL 是 18.3——更少层数,更好效果,且不需要 segment-level recurrence
- PG-19:22 层 Routing Transformer 33.2 perplexity,优于 36 层 Compressive Transformer 的 33.6——直接在 8192 长度上训练,而非用短序列训练再希望泛化到长序列
- 消融实验中 Random Transformer(3.076) < Local Transformer(3.009) < Routing Transformer(2.971),证实聚类内容的选择性是关键,而非单纯减少注意力范围

**适用边界**:论文指出 routing attention 比 local attention 慢约 1.2-1.7 倍(TPU 上稀疏操作支持不足),内存效率是首要目标,速度是次要的。在 PG-19 上,仅最后 2 层使用 routing head、其余层全用 local——说明长程依赖可能只需在高层捕获。

## 我能带走什么

以后遇到长序列建模任务,先不急着用固定窗口或全注意力,而先问:注意力的稀疏模式能不能由内容本身决定?如果序列中存在跨距离的语义依赖(同一实体重现、长程指代、主题一致性),routing 式的内容稀疏比位置稀疏更有优势;如果依赖主要是局部的(比如图像的局部纹理),local attention 已经够用且更快。

一个具体判断:当你在训练语言模型时发现 local attention 的 perplexity 不再随序列长度下降——说明模型需要看到更远的内容但看不到——这时候加 routing head 比加更多 local 层更有效,因为 routing 能把语义相关的远处 token 拉到一起,而 local 只是扩大窗口,代价是 O(nkd) 线性增长却未必命中关键依赖。