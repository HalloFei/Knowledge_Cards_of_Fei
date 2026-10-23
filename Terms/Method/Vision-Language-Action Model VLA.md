```json
{
  "term": "VLA",
  "full_form": "Vision-Language-Action Model",
  "folder": "Method",
  "definition": "以图像与语言为输入、直接输出机器人动作的端到端策略模型",
  "tags": ["robotics", "multimodal", "imitation-learning", "embodied-ai"],
  "aliases": ["视觉语言动作模型"],
  "storage_path": "Terms/Method/Vision-Language-Action Model VLA.md"
}
```

## 📖 详细定义

VLA 把"看图—听懂话—动手"合成一个模型：输入图像（常为多视角）+ 语言指令 + 本体状态（关节角、末端位姿），输出动作，端到端训练，骨干通常是一个预训练 VLM。

三个边界要分清：

- **不是"VLM + 规划器"**。那种是分层系统：VLM 输出文字计划，规划器再调用技能、低层控制器执行。VLA 的动作是模型**直接吐出来**的，中间没有文本瓶颈。
- **不是传统模仿学习策略**。普通 BC 只吃视觉、没有语言条件，也没有 web-scale 预训练；VLA 的泛化能力主要来自"预训练 VLM + 机器人数据微调"这个组合。
- **动作表示决定流派**：离散 token（RT-2 把动作当成词表里的词，和文本共享词表）、连续回归（OpenVLA 的 action head 直接回归 7 维动作）、生成式（π0 用 flow matching 生成动作块）。前者借语言模型的力，后者更适合高频、灵巧、多模态的动作分布。

真正的瓶颈是数据与本体：Open X-Embodiment 把 22 种机器人、百万级轨迹汇到一起，才有了跨本体训练的可能；但动作空间、相机外参、控制频率各不相同，归一化与对齐仍是工程大头。

## 实际应用

- RT-2 之后这条线快速产品化：Figure Helix、特斯拉 Optimus、Physical Intelligence 的 π0 / π0.5、Google 的 Gemini Robotics 系列。2026 年 7 月的 Gemini Robotics 2 进一步宣称全身控制——从步态到双手精细操作，并支持多机器人协同。
- 当前的现实分工是**分层**而非纯端到端：上半身操作与技能选择交给 VLA（数 Hz 到数十 Hz，取决于动作分块与模型大小），下半身平衡与步态仍由仿真里训练的 RL 控制器（Isaac Lab + PPO 一类）负责。所谓"一个模型控制全身"目前更接近"一个策略输出全身目标，底下还有一层控制器"。
- 竞争方案：经典运动规划 + 抓取检测在工业产线上仍更可靠、更可解释；"VLM 规划 + 低层技能库"在长程任务上仍占优。
- 已知瓶颈：推理频率与延迟、长程任务的误差累积、换本体后的泛化、以及安全边界（错了就是物理后果）。

## 🔬 前沿研究（3 项）

- `[2023] RT-2: Vision-Language-Action Models Transfer Web Knowledge to Robotic Control (arXiv:2307.15818)`：把动作离散成 token 与文本共用词表，用 web 数据与机器人数据联合微调，首次清晰展示"互联网知识可迁移到机器人控制"（检索关键词：RT-2 vision language action co-fine-tuning）。
- `[2024] OpenVLA: An Open-Source Vision-Language-Action Model (arXiv:2406.09246)`：7B 参数、基于 Open X-Embodiment 训练、LoRA 即可微调，成为社区事实基线，把 VLA 从"大厂演示"变成"可复现实验"（检索关键词：OpenVLA open-source 7B robot policy）。
- `[2025] Gemini Robotics: Bringing AI into the Physical World (arXiv:2503.20020)`：以 Gemini 为骨干的 VLA 与具身推理模型，强调灵巧操作、开放世界泛化与多本体迁移，是"通用机器人基础模型"路线最完整的公开材料之一（检索关键词：Gemini Robotics embodied reasoning VLA）。

## 🔗 相关概念（3 个）

- `VLM (Vision-Language Model)`：骨干/父概念——VLA 通常从 VLM 初始化，把输出头从"文本 token"换成"动作 token 或连续动作"。
- `模仿学习 (Imitation Learning / Behavior Cloning)`：训练范式——VLA 本质是大规模、带预训练的语言条件 BC，数据来自遥操作而非奖励试错。
- `世界模型 (World Model)`：互补或竞争——VLA 直接学策略，世界模型先学环境动力学，用于规划或生成合成数据（如宇树 UnifoLM-WMA 把视频生成模型当仿真器用）。

## ⚖️ 优缺点

- **优点**：① 语言条件 + 大规模预训练带来开放词汇指令理解与零样本泛化，换任务不必重写代码；② 端到端省掉手写感知管线与状态机，长尾场景的边际成本更低。
- **缺点**：① 数据昂贵（遥操作采集慢、贵、覆盖窄），跨本体迁移仍脆弱；② 推理延迟与长程误差累积，安全性与可验证性远不如经典控制栈，落地必须配安全层。
