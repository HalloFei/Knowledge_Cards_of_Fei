{
  "term": "语义VAD",
  "full_form": "Semantic Voice Activity Detection (语义语音活动检测)",
  "folder": "Concept",
  "definition": "基于语义信息判断音频中是否存在有效语音的检测技术",
  "tags": ["vad", "speech", "semantic", "asr"],
  "storage_path": "Terms/Concept/Semantic VAD.md"
}

## 📖 详细定义
语义VAD在传统VAD（仅依赖能量、过零率等声学特征判断有声/无声）基础上，进一步引入语义层面的理解能力。它不仅检测"有没有人说话"，还能判断语音内容是否有效或相关——例如区分正常对话与背景噪声中的模糊人声，或过滤无意义呓语。

## 实际应用
语义VAD广泛应用于智能客服、会议记录、语音助手等实时场景，是端到端语音交互管线的核心前置模块。当前主流方案仍以声学VAD（如 WebRTC VAD、Silero VAD）为主，但语义VAD在大模型时代正快速崛起——结合ASR与LLM语义判断的混合方案逐渐成为高端场景的新趋势。竞争方案包括基于能量阈值的传统VAD和基于深度学习的端到端VAD。

## 🔬 前沿研究（2 项）
- `[2024] SemanticVAD: LLM-Aware Voice Activity Detection`：将大语言模型的语义理解能力融入VAD，显著降低误判率。（检索关键词：semantic VAD LLM）
- `[2023] Context-Aware VAD with Semantic Embeddings`：利用预训练语义嵌入提升VAD在嘈杂环境下的鲁棒性。（检索关键词：context-aware VAD embedding）

## 🔗 相关概念（3 个）
- `VAD（Voice Activity Detection）`：父概念。语义VAD是传统VAD的语义增强版，传统VAD仅做声学层面的有声/无声二分类。
- `ASR（Automatic Speech Recognition）`：互补关系。语义VAD常作为ASR的前置模块，过滤非有效语音以提升识别准确率与效率。
- `Endpoint Detection`：易混淆概念。端点检测关注语音段的起止边界，语义VAD更关注语音内容的语义有效性。

## ⚖️ 优缺点
- **优点**：1) 能过滤声学VAD无法区分的"有声但无意义"片段，减少下游误识别；2) 在嘈杂环境下鲁棒性更强。
- **缺点**：1) 引入语义模型后计算开销显著增大，实时性受限；2) 依赖额外ASR/LLM组件，系统复杂度和延迟增加。