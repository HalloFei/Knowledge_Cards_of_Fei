---
title: 提示词自动优化器的分类学——记忆、信号与真正的优化目标
type: concept
date: 2026-08-31
tags: [concept, prompt-engineering, optimization, evolution]
related:
  - notes/20260831T120000--paper-diverse-prompts-map-elites__paper.md
  - notes/20260826T120000--paper-gepa__paper.md
  - notes/20260831T150000--paper-apo-gradient-descent-beam-search__paper.md
---

# 提示词自动优化器的分类学

**副标题**：喂什么进 LLM、看什么当反馈、真正在爬什么坡

## 速读

同一片田里的**七个**工作——APE、OPRO、EvoPrompt、PromptBreeder、**APO/ProTeGi**、Diverse Prompts、GEPA——都在做"把写提示词变成搜索问题"，但**它们的差别几乎全在三件事上**：(1) 每次调用 LLM 时喂进去的历史是什么形态；(2) 外部反馈是标量还是文本；(3) LLM 在这个循环里扮演什么角色（拟合器 / 变异算子 / 反思器）。

一条容易被忽视的观察：**"记忆容量"和"信号质量"是替代关系**——历史越贫瘠，你需要塞进越多历史；历史越有信息量，你需要塞进的越少。OPRO 塞 20 条(指令, 分数)才能勉强外推；APO 塞 1 个 minibatch 的错例 + 一段文本诊断，但**当前 minibatch 一用完就丢**；GEPA 塞 1 个 minibatch + 文字诊断 + Pareto 前沿 + 演化树。这条替代律解释了 GEPA "35× 样本效率"数字的一大半来源——不是算法更聪明，而是**每次 rollout 携带的可推理信息多得多，而且跨轮积累**。APO 卡在 "信号已经升级为自然语言但记忆还是马尔可夫" 的中间态，这正是它 3 步就过拟合、比 GEPA 少一个数量级效率的根本原因。

## 它到底在解决什么

**背景问题**：提示词空间是离散、无梯度、组合爆炸的。人手调 prompt 效率低；用 RL 更新权重成本高。中间地带就是"用 LLM 自己当优化器"——但"当优化器"这个词遮蔽了一整层设计选择。

**核心分歧**：LLM 本身**无状态**，每次调用都是全新的。所以"这个方法记得之前的提示词"这句话是误导——真正记忆的是**外层框架**，它决定 LLM 在本次调用能看到多少历史、什么形态的历史。把"记忆"拆开有三种载体：

| 载体 | 形态 | 谁在用 |
|---|---|---|
| 上下文窗口 | 把历史(prompt, score)拼进当前调用 | OPRO |
| 种群池 | 显式数据结构，每代挑父代 | EvoPrompt / PromptBreeder / Diverse Prompts |
| 演化树 + Pareto 前沿 | 每候选记祖先 + "在哪些样本上强" | GEPA |

**分歧的第二层**：反馈信号的类型决定 LLM 能扮演什么角色。标量信号只能被外层的选择算法用；自然语言反馈可以被 LLM 本身**读进上下文**，做出针对性修改——这就把"变异"从盲变升级为"读诊断写补丁"。

## 它真正看见了什么

**逐个拆开**（我把每种方法的 meta-prompt 骨架列出来，让"喂什么"这件事具体化）：

### OPRO——LLM 当拟合器

Meta-prompt 骨架：
```
Below are some previous instructions and their scores:
  instruction: "Let's think step by step" → score: 71.8
  instruction: "Break this problem down"   → score: 68.4
  ...
Write a new instruction that is different and achieves a higher score.
```
喂给 LLM 的是 K≈20 条(指令, 分数)对，按分数升序排列（利用位置偏差让高分靠近生成位置）。信号是纯标量。真正在做的事：**让 LLM 读一段响应曲面样本，做类比外推**。爬坡能力完全取决于"分数曲线是否在语义上可归纳"——如果指令间的分数差看起来像随机噪声，LLM 也归纳不出方向。

### EvoPrompt——LLM 当橡皮泥

Meta-prompt 骨架：给两个父代提示词，要求 LLM "识别差异部分、变异、和公共部分拼接"。**不喂分数、不喂反馈**。信号（标量）留在外层做锦标赛选择。LLM 只负责"变异"，不负责"判断方向"。这就是经典遗传算法框架里 LLM 替换位翻转的做法——LLM 是**语义邻域内的随机游走算子**。

### PromptBreeder——让进化学会自己的变异

在 EvoPrompt 上加了递归层：**两个种群同步进化**——任务提示词种群 P_task，加变异提示词种群 P_mut（形如"请更简洁"、"请加反例"）。每次执行 `m(t)` 得到新任务提示词；孩子的分数**反向回溯**给它的变异父母，让适配当前任务的变异器被选择性保留。这是自指结构——**系统在学习"如何为自己找到好变异"**，是这一族里唯一试图学习优化算子本身的工作。

### APO/ProTeGi——LLM 当写检讨的学生

见 [APO 笔记](20260831T150000--paper-apo-gradient-descent-beam-search__paper.md)。Meta-prompt 骨架有两段：**∇ 段**——`My current prompt is X, it gets these examples wrong Y, give reasons why`——让 LLM 生成"文本梯度"（对失败原因的自然语言批评）；**δ 段**——`Based on the problem that {gradient}, write improved prompts`——让 LLM 读着批评把 prompt 往反方向改。信号从 scalar 升级为 natural language critique，LLM 首次在这一族里承担了**部分信用分配**——它读错例、写诊断，方向不再来自随机扰动而来自语义判断。外层是 beam search + bandit best-arm identification（UCB 系列）压 API 预算。**关键短板**：诊断只看当前 minibatch，**历史不进下一轮上下文**——信用分配是马尔可夫的，所以 Figure 4 里 3 步就见顶过拟合。APO 是这一族里第一个把 LLM 用在信用分配上的工作，但只做了一半——它有诊断但没有历史。

### Diverse Prompts——LLM 完全不在优化循环里

见 [MAP-Elites 笔记](20260831T120000--paper-diverse-prompts-map-elites__paper.md)。LLM 只被用来**执行**提示词、不参与优化；真正的优化器是外层 MAP-Elites 分格器，输入信号只有 `f = accuracy@50`。所以它照亮的是"CFG 采样分布 × 单目标准确率"投影出来的形状，而非提示词空间本身的性能地形。

### GEPA——LLM 当读错题本的学生

见 [GEPA 笔记](20260826T120000--paper-gepa__paper.md)。Meta-prompt 骨架（简化附录 C）：把当前提示 + 一小批 rollout 的输入/输出/**文字诊断**喂进去，让 LLM "识别当前提示缺失什么、抽取利基事实性知识、总结可泛化策略、写新提示"。信号从数字升级为一段可推理的文本；标量分数只用于"是否加入 Pareto 前沿"，**不再指导变异方向**。这是这一族里唯一让 LLM 真正做**信用分配**的工作——分配是靠语言语义天然带出来的，不是靠优化算法算出来的。

**综合对比**：

| | 喂进 LLM 的内容 | 外部反馈信号 | LLM 扮演的角色 | 真正在爬的坡 |
|---|---|---|---|---|
| **APE** | (input, output)示例对 | 标量 | 逆向工程师 | 单点最优 |
| **OPRO** | K 条(指令, 分数) | 标量 | 元优化器（读曲线） | 分数曲线的语义外推 |
| **EvoPrompt** | 2 个父代（无分数） | 标量（选择用） | 变异算子（盲变） | 种群平均分 |
| **PromptBreeder** | 父代 + 变异指令 | 标量（双回路） | 变异算子 + 元变异算子 | 任务分 × 变异器适应度 |
| **APO/ProTeGi** | 当前 prompt + 错例 + **文本梯度** | 自然语言（∇段）+ 标量（bandit筛选） | 伪梯度生成器 + 反梯度编辑器 | 文本空间的近似梯度下降（马尔可夫） |
| **Diverse Prompts** | 无历史，CFG 随机采样 | 标量 | 无（只当解题者） | 表型格子覆盖率 |
| **GEPA** | 提示 + rollout + **文字诊断** | 自然语言 + 标量 | 反思器（读诊断写补丁） | 每模块局部质量 + Pareto 覆盖 |

## 我能带走什么

**判断规则**（下次评估任何"用 LLM 优化 X"的工作时用这条）：**看它是否把 LLM 的语言理解能力真正用在了信用分配上，而不是仅仅当作生成器。** 只把 LLM 当变异算子的方法（EvoPrompt 家族）本质上是把 LLM 塞进遗传算法当花瓶——LLM 在这里可以被同义词字典替换，性能差别只在变异步的语义连续性。真正利用 LLM 独特能力的是 GEPA 那种"读诊断写补丁"——诊断信息只有会读自然语言的模型能利用。

**选型清单**（把这套框架拿来用）：
1. 我的环境**能吐出结构化文字反馈**吗？能——GEPA 类反思式方法优先；不能——退回 OPRO/EvoPrompt。
2. 我要"最优的一条"还是"结构分布图"？前者用 GEPA/APE，后者用 Diverse Prompts。
3. 我的候选池**多样性从哪里来**？维护 top-k 几乎必然陷入局部最优；Pareto 前沿（GEPA）或表型分格（MAP-Elites）是低成本的多样性来源。
4. 我优化的是**提示还是权重**？前者选本类方法，后者才动 RL/LoRA——这两条路是替代品，不是补充品。

**一条能预测的推论**：反馈信号越丰富，反思式方法（GEPA）相对进化式方法（EvoPrompt）的优势越大；反馈越贫瘠（纯 0/1，无解释），两者差距缩水，甚至进化式方法因为不做无用的"读诊断"步骤反而更省算力。这条预测可以用来判断未来任何"XX-GEPA-like"工作是否符合它的适用边界。

**一个反直觉的观察**：Diverse Prompts 里 LLM **根本没参与优化**——它只是执行者。这说明"提示词自动优化"这个标签下的工作，甚至可以完全不用 LLM 做优化器。判断一个方法值不值得用 LLM 做优化器，只有一个问题：**你手里的信号里有 LLM 才能利用的东西吗？** 没有——用遗传算法或贝叶斯优化更省。