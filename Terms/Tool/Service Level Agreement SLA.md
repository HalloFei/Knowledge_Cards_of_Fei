{
  "term": "SLA",
  "full_form": "Service Level Agreement",
  "folder": "Concept",
  "definition": "服务提供方与用户约定的服务质量与可用性保障协议",
  "tags": ["sla", "reliability", "ops", "contract"],
  "storage_path": "Terms/Concept/Service Level Agreement SLA.md"
}

## 📖 详细定义
SLA（服务等级协议）是服务方与使用方之间的正式契约，明确可用性、响应时延、故障恢复时间（RTO/RPO）等量化指标，以及未达标时的补偿或降级方案。在软件/云服务语境下，SLA 是衡量服务可靠性的核心承诺。

## 实际应用
在当前云服务与 SaaS 领域，SLA 是主流且不可或缺的合规要求。AWS、Azure、阿里云等公有云普遍承诺 99.9%~99.99% 月度可用性；LLM API 服务（OpenAI、Anthropic）也开始提供企业级 SLA。对于边缘设备/AI 硬件产品，常见做法是补充「云端可用性目标 + 断网降级可用时长」双指标，避免云依赖成为单点故障。SLO/SLI 体系（Google SRE 提出）是 SLA 的工程化延伸，属当前最佳实践。

## 🔬 前沿研究（2 项）
- `[2024] SLO-aware LLM Serving`：研究在延迟 SLO 约束下动态调度 LLM 推理请求（关键词：SLO-aware inference scheduling）。
- `[2025] Probabilistic SLA for AI Agents`：探索面向不确定性输出的 Agent 概率型 SLA 定义（关键词：probabilistic SLA, agent reliability）。

## 🔗 相关概念（3 个）
- `SLO (Service Level Objective)`：SLA 的内部量化目标，通常严于 SLA 对外承诺值（父/子关系）。
- `SLI (Service Level Indicator)`：具体测量指标（如成功率、P99 延迟），是 SLO 的观测基础（易混淆：SLI 是测量，SLO 是目标，SLA 是承诺）。
- `MTBF/MTTR`：可靠性统计指标，与 SLA 可用性目标互补，用于内部工程评估。

## ⚖️ 优缺点
- **优点**：
  1. 明确责任边界，为用户提供可预期的服务质量保障。
  2. 驱动运维体系建设（监控、告警、容量规划）标准化。
- **缺点**：
  1. 过高的 SLA 承诺会显著推高架构冗余与运维成本。
  2. 补偿条款通常远低于业务实际损失，象征意义大于赔付意义。