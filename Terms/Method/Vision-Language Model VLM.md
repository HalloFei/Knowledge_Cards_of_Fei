```json
{
  "term": "VLM",
  "full_form": "Vision-Language Model",
  "folder": "Method",
  "definition": "同时接受图像与文本、并在两者之间做联合推理的模型",
  "tags": ["multimodal", "vision", "llm", "contrastive-learning"],
  "aliases": ["视觉语言模型"],
  "storage_path": "Terms/Method/Vision-Language Model VLM.md"
}
```

## 📖 详细定义

VLM 是能同时吃图像和文本、并在两者之间推理的模型。技术上分两支，差别不在规模，在**输出是什么**：

- **对齐式（CLIP 系）**：图像塔与文本塔各自编码，用图文对的对比学习（InfoNCE）把两者拉进同一个嵌入空间。它产出的是"相似度"，不是"句子"——所以只擅长检索、零样本分类、rerank，不能生成描述。
- **生成式（LLaVA / BLIP-2 / Flamingo 系）**：视觉编码器 → 投影层或重采样器 → LLM，图像被压成若干视觉 token 拼进语言上下文，"看图回答"于是退化成 next-token prediction。今天说 VLM 通常指这一支。

三个必须分清的边界：VLM ≠ 图像分类器（输出不是固定标签，是开放文本）；VLM ≠ 纯视觉模型（必须有语言接口）；VLM ≠ VLA（VLM 输出文本，VLA 直接输出动作，见相关概念）。

核心工程矛盾是**分辨率 vs token 预算**：图越大看得越细，但视觉 token 会线性吃掉上下文窗口与延迟预算，这也是动态分辨率、token 压缩一类技巧存在的理由。

## 实际应用

- 已经是产品级能力，不是实验室玩具：文档/票据/表格理解、OCR 后处理、工业质检报告生成、医疗影像初筛、GUI agent 的屏幕理解、自动驾驶场景问答与 corner case 标注。
- 开源可私有部署的强基线：Qwen2.5-VL、InternVL 系列；闭源侧是 GPT-4V / Gemini 一档。选型时先看**是否需要在私有数据上微调**，再比榜单。
- 竞争方案：窄任务上专用检测/分类小模型更便宜更快；结构化文档上"OCR + 纯文本 LLM"的管线仍可能更划算。VLM 的性价比来自"一个模型覆盖多任务"，不是单点最优。
- 已知弱项：细粒度计数、精确空间关系、小字与密集表格、以及描述不存在的物体（幻觉）。

## 🔬 前沿研究（3 项）

- `[2021] Learning Transferable Visual Models From Natural Language Supervision (CLIP, arXiv:2103.00020)`：4 亿图文对上的双塔对比学习，零样本即接近有监督 ResNet，奠定"图文共享嵌入"范式，至今仍是大多数 VLM 的视觉编码器来源（检索关键词：CLIP contrastive language image pretraining）。
- `[2023] Visual Instruction Tuning (LLaVA, arXiv:2304.08485)`：用 GPT-4 把图像转成指令数据，视觉 token 直接接进 LLM 做指令微调，确立"视觉编码器 + 线性投影 + LLM"的开源标配架构，后续 LLaVA-1.5 / NeXT 沿此线扩展（检索关键词：visual instruction tuning LLaVA）。
- `[2025] Qwen2.5-VL Technical Report (arXiv:2502.13923)`：原生动态分辨率 + 绝对时间编码，把长视频的时序定位纳入统一框架，是目前开源侧最强的可部署基线之一（检索关键词：Qwen2.5-VL dynamic resolution video grounding）。

## 🔗 相关概念（3 个）

- `VLA (Vision-Language-Action Model)`：下游延伸——VLA 通常以 VLM 为骨干，把输出从文本换成动作；区别落在动作空间、控制频率与安全约束上。
- `CLIP`：VLM 的对齐基石，提供图文相似度与视觉编码器；对比学习路线与生成式路线正是在这里分叉。
- `多模态大模型 (LMM)`：上位/近义概念——LMM 可含音频、视频、传感器等更多模态，VLM 特指"视觉 + 语言"这一支，二者常被混用。

## ⚖️ 优缺点

- **优点**：① 自然语言接口，零样本/少样本迁移强，一个模型覆盖描述、问答、定位、文档解析等多种任务；② 可直接复用 LLM 的世界知识与推理能力，不必为每个任务重训。
- **缺点**：① 幻觉与细粒度感知（计数、空间关系、小字）不可靠，高风险场景必须加校验或专用模型；② 高分辨率输入带来 token 与延迟成本，实时或端侧部署受限。
