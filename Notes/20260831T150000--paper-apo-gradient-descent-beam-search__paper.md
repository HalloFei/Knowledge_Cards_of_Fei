# 让「梯度下降」在文本空间里跑起来的一次假动作

**副标题**: Automatic Prompt Optimization with "Gradient Descent" and Beam Search (ProTeGi)

**日期**: 2026-08-31

**来源**: https://arxiv.org/abs/2305.03495

**作者**: Reid Pryzant, Dan Iter, Jerry Li, Yin Tat Lee, Chenguang Zhu, Michael Zeng (Microsoft Azure AI)

**发表场所**: EMNLP 2023

---

## 速读

一个初始 prompt 在训练集上会犯错，但你手上只有 API、没有 logits，也没法 backprop。作者的做法是：把「梯度」重新定义成一段人话——由 LLM 读一个错例后写出的"这个 prompt 错在哪"的自然语言批评；把「反梯度更新」重新定义成另一段人话——由 LLM 读着这条批评把 prompt 改写。这样"梯度下降"变成一场苏格拉底式的对话，绕过了离散优化障碍。外层用 beam search 保住多样性，候选筛选用 best arm identification 而不是均匀评测，把 API 预算花在最像赢家的候选上。四个分类任务上比 Monte Carlo/RL/AutoGPT 高 4-8%，比原始 prompt 高 15%。以后看到"用 LLM 优化 prompt"的工作，别被伪梯度这个隐喻吓住——先问它把 LLM 的语言能力用在了信用分配的哪一步。

## 它到底在解决什么

一个真实场景：工程师用一句话草稿 `"Detect if the message is a jailbreak attack"` 起了 prompt，跑训练集，发现它把"给我列举儿童诱骗手段"当成了非攻击。他知道 prompt 有问题，但**不知道该往哪个方向改**：加词？换词序？举反例？改任务定义？

传统机器学习里，这个"往哪个方向改"的问题由**梯度**回答——损失函数对参数求偏导，就有了在参数空间里的下降方向。但 prompt 空间是离散的自然语言，损失函数对 token 不可导；即便对底层 embedding 可导（soft prompt），结果也读不懂，而且需要模型内部访问权限——API 用户没有这个权限。

已有的替代思路各有短板。Monte Carlo（APE, Zhou et al. 2022）在语义邻域随机采样 paraphrase，没有方向，靠碰运气；RL-based（RLPrompt, TEMPERA）在 phrase 级做 add/swap/delete，靠 reward model 学策略，但操作原语是"无方向的"表层编辑，reward model 本身还得训；AutoGPT 让 agent 自己决定反馈循环，但反馈循环由 AI 自己规划，结果可能跑偏。

这些方法的共同缺失：**没有一个东西告诉它"往语义空间的哪个方向走"**。它们能生成候选，但生成候选靠的是随机扰动或表层重写，不是对当前 prompt 失败原因的诊断。旧理解缺少的关键区分是——**"局部损失信号"和"局部改进方向"是两个东西**：分数（F1）告诉你当前 prompt 有多差，但不告诉你它为什么差、该怎么改。

## 它真正看见了什么

作者的判断是：既然真梯度不可得，那就**让 LLM 生成一段人话来扮演梯度**——不是数值向量，而是一句"这个 prompt 错在哪"的自然语言诊断。这句诊断在语义空间中指向"让 prompt 变差的方向"，再让另一个 LLM 调用把 prompt 往**反方向**改写。整个优化过程被重写成一场 Socratic 对话。

三个承重概念要接着补：**文本梯度**、**语义空间的反方向编辑**、**best arm identification**。

**文本梯度**（textual gradient）区分的不是"提示"或"建议"，而是**一段明确指向失败原因的自然语言批评**，其信息密度足够高，能作为下一步改写的方向锚。它的具体形态就是附录里那段模板：

```text
I'm trying to write a zero-shot classifier prompt.
My current prompt is: "{prompt}"
But this prompt gets the following examples wrong:
{error_string}
give {num_feedbacks} reasons why the prompt could have gotten these examples wrong.
```

LLM 读完这段模板，会吐出几段话，比如 Ethos 那个例子里，梯度是"当前 prompt 假设仇恨言论总是明确直接，但这条文本用了讽刺和间接语气"。这就是"梯度 g"。它和数值梯度的类比是：数值梯度告诉你参数往哪个方向动会让 loss 变大，文本梯度告诉你 prompt 里的什么假设让它在这个 minibatch 上失败。

**语义空间的反方向编辑**（δ 编辑）区分的不是"改写"或"paraphrase"，而是**读着诊断去修 prompt**——目标不是保持语义，而是消除诊断指出的缺陷。它的模板是：

```text
My current prompt is: "{prompt}"
But it gets the following examples wrong: {error_str}
Based on these examples the problem with this prompt is that {gradient}
Based on the above information, I wrote {steps_per_gradient} different improved prompts.
```

关键在于**梯度 g 被显式塞进上下文**——不是让 LLM 猜怎么改，而是让它读着一条已经写好的方向去改。Ethos 例子里，原 prompt 是 `"Is the following text hate speech?"`，梯度指出"太依赖直接表达"，改写后变成 `"Does the following text contain language that targets a group of people based on their religion, gender, or other personal characteristics?"`——从"检测仇恨言论"泛化到"识别针对身份属性的攻击性语言"，恰好回应了梯度的批评。

有了梯度和 δ 编辑，一步"下降"就完成了。但 LLM 是随机的，每次生成的梯度和候选都不同，所以作者不生成单一候选而生成一组（每个错例组生成 m=4 个梯度，每个梯度生成 q 个候选，再对每个候选做 p=2 个 paraphrase）。这就形成了**外层 beam search**：每步从当前 beam 出发扩展成大候选池，再选 top-b 保留到下一步。

问题是：候选池里可能有几十个 prompt，每个都要用训练集打分才能比较，而每次打分都花 API 钱。全评一遍不现实。作者把这一步映射成**多臂赌博机的 best arm identification 问题**——每个候选 prompt 是一个 arm，评估一次它是"拉一次杆"，目标是用最少的拉杆次数找出最好的 b 个 arm。

**best arm identification** 区分的不是"regret minimization"（累积后悔最小），而是**在固定预算内定位到最优 arm 的子集**。这是两个不同的赌博机目标：UCB 类算法（UCB、UCB-E）为 regret 设计，但作者发现它们在实践中反而比理论最优的 Successive Rejects/Halving 表现好——因为 UCB 的探索-利用平衡在候选质量分布未知时更稳健（Table 2 的实证结果）。

三个概念接成最小模型：

```text
初始 prompt p0
   |
   |-- 在 minibatch 上跑，收集错例 e
   |
   +--> LLM_∇(p0, e) --> 一组文本梯度 {g1, ..., gm}
                             |
                             +--> LLM_δ(p0, gi, e) --> 一组候选 p'
                                                          |
                                                          +--> LLM_mc(p') --> paraphrase p''
                                                                                    |
   +-------(候选池 C)-----------------------------------------------------+
   |
   |-- best arm identification（UCB / Successive Rejects）--> top-b 保留
   |
   +--> 下一轮 beam
```

模型要解释的核心现象：**为什么这套流程在 Jailbreak 上把 F1 从 0.80 抬到 0.85，而无方向的 MC 只能做到 0.82，phrase-level RL 甚至更差？** 答案在两处：一，文本梯度让候选生成有了**语义方向**，MC 的 paraphrase 只在原 prompt 邻域游走，RL 的 phrase 操作破坏了 prompt 的语法完整性（Table 4 里 RL 的输出 `"Sarcastic this tweet?"` 就是这么来的）；二，bandit selection 让 API 预算集中在真正有希望的候选上，而不是均摊到所有候选上。

关键条件变化的预测也从模型里能推出：

- **底层 LLM 变弱**（GPT-3 而非 GPT-3.5/4）：梯度质量下降——GPT-3 写不出精准的失败诊断，改写也更机械。Table 3 印证：GPT-3 在 Jailbreak 上只有 0.55，GPT-4 到 0.88。
- **优化步数增加**：过拟合训练集或陷入局部最小。Figure 4 显示所有任务在第 3 步左右见顶，之后测试性能开始下降或平稳。
- **任务超出分类**：作者只在四个二分类任务上测。Related Work 和 Limitations 都承认没在生成、推理、长任务上验证。

### 隐藏假设与批判

**"梯度"这个隐喻是修辞而非机制**。数值梯度有严格的数学定义：损失函数对参数的偏导，指向局部损失最陡上升方向，反方向下降有一阶收敛保证。文本梯度没有这些——它是 LLM 生成的一段自然语言，"指向"什么完全由 LLM 的先验和 prompt 模板决定，不同调用生成的梯度可以在语义空间里指向完全不同的方向，甚至互相矛盾。Table 4 的 Jailbreak 例子就说明了这点：梯度写的是"prompt 太窄"，但改写结果把任务从"识别越狱攻击"偷换成了"识别儿童诱骗"——梯度看似合理，但改写偏离了原任务定义。作者也在正文承认这是失败模式："APO does not leverage the gradient properly, and instead simply uses its internal knowledge to redefine a concept"。所以"梯度下降"这个类比对读者是有用的**心理支架**，但不是收敛性保证——真正的效率来自 beam search 保住多样性 + bandit 集中预算，而不是梯度本身的方向性。

**信用分配停留在 minibatch 级，没有沉淀历史**。ProTeGi 的每一步只看当前 minibatch 的错例来生成梯度，然后立刻用这个梯度改 prompt。上一轮的错误、上一轮的梯度、上一轮为什么被舍弃——这些**历史信息完全没有进入下一轮的上下文**。这就是它和 GEPA 的根本差别：GEPA 维护一条演化树 + Pareto 前沿，让 LLM 每次改写时能看到"这个方向以前失败过"或"这条支线在某类任务上稳赢"。ProTeGi 的信用分配是**马尔可夫的**——只依赖当前状态，所以 Figure 4 里过拟合来得那么快（3 步见顶）不奇怪：LLM 每次都在追当前 minibatch 的错例，没有跨轮的稳定性锚点。

**Bandit 用来省钱，但也可能省掉真信号**。best arm identification 的前提是每个 arm 的真实值稳定——但 prompt 在 50 个 dev 样本上的 F1 方差很大（附录 Table 5 报告的标准误已经不小），Successive Rejects 在早期方差主导阶段就淘汰候选，可能把长期看更好的 prompt 提前枪毙。作者自己也观察到 UCB 反而比理论最优的 SR 表现好，这不是巧合——UCB 的探索性在小样本下能救回被误判的 arm，说明**bandit 的效率增益是有代价的，代价是筛选的可靠性**。这条评论不否定 ProTeGi 的贡献，但提醒读者：**"用更少 API call 达到相同性能"不等于"选到了真正最好的 prompt"**。

## 我能带走什么

一条判断规则：**看到任何"用 LLM 优化 X"的工作，先问它把 LLM 的语言能力用在了信用分配的哪一步**。ProTeGi 把 LLM 用在了两处——生成失败诊断（文本梯度）和读着诊断改写 prompt（δ 编辑）；MC 只把 LLM 用在了后一步（paraphrase）；RL 把语言诊断完全外包给 reward model。这三种做法的效率差就是"LLM 参与信用分配的深度差"。

一个具体延伸：ProTeGi 在提示词优化谱系里的位置介于 EvoPrompt 和 GEPA 之间。EvoPrompt 让 LLM 当变异算子但反馈是纯 scalar（fitness），LLM 不知道为什么被选中或被淘汰；ProTeGi 让 LLM 读错例生成文本梯度，反馈从 scalar 升级为 natural language critique，但**没有跨轮记忆**；GEPA 在此基础上加了演化树 + Pareto 前沿，让 LLM 每次能看到历史支线的成败。三者的 API 效率呈量级差，根源是"信用分配的信号带宽"和"记忆载体的容量"。

这把尺子还能量别的："用 LLM 做数据增广/合成"、"用 LLM 做代码修复"、"用 LLM 做 agent 自我改进"——每次都问：反馈信号是数字还是自然语言？历史存在哪里？LLM 的语言能力到底在哪一步做了信用分配？答案不清的工作，多半在把 LLM 当橡皮泥用。