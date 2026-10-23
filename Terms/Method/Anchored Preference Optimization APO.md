{
  "term": "APO",
  "full_form": "Anchored Preference Optimization",
  "folder": "Method",
  "definition": "通过显式锚定模型与偏好数据关系来解决对齐中规范不足问题的可控对齐目标族",
  "tags": ["alignment", "llm", "preference-optimization", "contrastive-learning"],
  "aliases": ["锚定偏好优化"],
  "storage_path": "Terms/Method/Anchored Preference Optimization APO.md"
}

## 📖 详细定义
APO 是 2024 年由 imec-根特大学、斯坦福大学和 Contextual AI 联合提出的一系列对齐目标，核心解决 LLM 对齐中的 **under-specification（规范不足）** 问题——即偏好数据与训练目标之间关系不明确导致的学习信号不一致。APO 包含两个变体：**APO-zero**（增大获胜输出概率、降低失败输出概率）和 **APO-down**（同时降低两者概率），通过显式控制模型概率分布变化方向提供比 DPO 更细粒度的训练控制。配套的 **CLAIR**（Contrastive Learning from AI Revisions）数据方法通过最小但有意义的对比修订生成偏好对，确保偏好信号精准。在 Llama-3-8B-Instruct 上，CLAIR + APO 在 MixEval-Hard 上提升 7.65%，将 GPT4-turbo 差距缩小 45%。

## 实际应用
APO 定位于 LLM 训练流程的 **SFT 之后的对齐阶段**，是 DPO/RLHF 的替代或增强方案。当前开源社区（Contextual AI）已发布代码实现，但尚未达到 DPO 的广泛采用程度。其 CLAIR 数据创建方法可与现有 SFT 流程结合：在 SFT 模型基础上，用 CLAIR 对模型输出做最小修订生成偏好对，再用 APO 目标进行对齐训练。与 ORPO（将 SFT 和对齐合并为单阶段）不同，APO 保持 SFT 与对齐阶段的分离，提供更独立的控制。竞争方案包括 DPO、KTO、SimPO、SPPO 等。

## 🔬 前沿研究（2 项）
- `[2024] Anchored Preference Optimization and Contrastive Revisions (arXiv:2408.06266)`：APO 原论文，首次提出 APO 对齐目标族和 CLAIR 数据创建方法，在多个基准上验证了可控对齐目标优于传统 DPO（检索关键词：anchored preference optimization underspecification）。
- `[2024] Controllable Preference Optimization (CPO, arXiv:2402.19085)`：与 APO 同期提出的可控多目标对齐方法，通过可训练的控制向量调节 helpfulness/harmlessness 等维度权重，与 APO 的控制粒度思路互补（检索关键词：controllable preference optimization multi-objective）。

## 🔗 相关概念（3 个）
- `DPO (Direct Preference Optimization)`：直接竞争/父概念——APO 在 DPO 基础上增加了对模型概率分布变化的显式锚定控制，解决 DPO 中偏好信号与优化目标关系不明确的问题。
- `CLAIR (Contrastive Learning from AI Revisions)`：配套数据方法——通过对模型输出做最小但有意义的 AI 修订生成偏好对，为 APO 提供高对比度训练数据；CLAIR 可与任何对齐目标配合使用。
- `ORPO (Odds Ratio Preference Optimization)`：易混淆——同为 SFT+对齐的优化方法，但 ORPO 将 SFT 损失和对齐损失合并为单阶段训练，APO 保持两阶段分离并专注于对齐目标的控制精度。

## ⚖️ 优缺点
- **优点**：① 显式控制模型概率变化方向，训练更稳定且可解释；② CLAIR 数据方法确保偏好对具有最小但有意意义的对比，提升学习信号质量
- **缺点**：① 相比 DPO 额外引入锚定参数，实现复杂度略有增加；② 2024 年刚提出，社区生态（如 HuggingFace TRL 集成）不如 DPO 成熟