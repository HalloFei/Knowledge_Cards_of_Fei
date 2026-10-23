```json
{
  "term": "DeepInfer",
  "full_form": "DeepInfer",
  "folder": "Tool",
  "definition": "DeepSeek 开源的生产级大模型推理框架",
  "tags": ["inference-framework", "deepseek", "moe", "serving"],
  "aliases": ["DeepInfer 推理框架"],
  "storage_path": "Terms/Tool/DeepInfer.md"
}
```

## 📖 详细定义

DeepSeek 生产推理系统（支撑 V4.1-Flash）的开源版本，Apache 2.0。定位非常克制：**只支持 DeepSeek-V4.1-Flash 一个模型**，硬件限定 NVIDIA SM100 与华为 Ascend 950。

血统要说清楚：骨架源自 vLLM（多个源文件头标注 `Derived from vLLM; modified by DeepSeek`），异步调度参考 SGLang / NanoFlow，kernel 用自家 FlashMLA / DeepGEMM / DeepEP / TileKernels / DeepSelect 加 TileLang 自定义算子。

特性清单：优化 kernel、Attention DP / 平衡 CP、MoE EP + EPLB、DSpark 投机 + confidence 调度、SWA bounded replay、零开销异步调度、batch-invariant 推理、确定性采样、结构化输出。

**没有开源的部分**同样重要：KV cache 存储系统、P/D 分离、跨副本负载均衡。所以拿它复现不了完整生产系统，只能做离线推理。

与通用框架的根本差别：它是**为单一模型特化**的。因为知道「后段 KV 是一次投影」「consumer SWA 反正要重算」，才敢做 bounded replay 这种通用框架做不了的优化。

## 实际应用

- 适合：研究 / 复现 DeepSeek 生产推理的具体做法；在自有集群部署 V4.1；借鉴确定性推理与 SWA 优化的实现细节。
- 不适合：要跑 Llama / Qwen 等多模型（用 vLLM / SGLang）；要开箱即用的 P/D 分离与负载均衡（需自建）。
- 竞争格局：vLLM（通用、生态最大）、SGLang（radix cache + 高性能调度）、TensorRT-LLM（NVIDIA 编译期特化）。DeepInfer 的差异化不在通用性，而在「模型-框架 co-design 的深度」。
- 生产数据（官方 Q&A）：单集群 155 万亿 token/天、$38/GPU-hour（谷时电价）；decode 用 EP64-128 的 Wide-EP，prefill 用 EP2-8。

## 🔬 前沿研究（3 项）

- `[2026] DeepInfer 仓库与部署 Q&A`（本工作区可对照 `doc/inference_deployment_qa.md`）：给出架构、EPLB、KV 分层存储与生产指标，是理解其设计取舍的一手材料。
- `[2026] DeepSeek-V4 技术报告`（本工作区 Papers/ 下）：模型侧的 CED / 稀疏注意力 / 压缩设计，决定了框架侧能做什么优化。
- `[2026] DeepSeek Recipe`（配套开源）：API 协议与 token 编解码，保证第三方部署的 API 行为与官方一致。

## 🔗 相关概念（3 个）

- `vLLM`：血统来源与主要竞品——通用性强、生态大，但难以做模型级特化。
- `SGLang`：调度与确定性推理的同类实现，radix cache 是其标志。
- `TileLang`：DeepInfer 自定义 kernel 的主要 DSL，也是 DeepSeek 此前开源的算子栈。

## ⚖️ 优缺点

- **优点**：① 生产验证过的完整优化组合（EPLB、DSpark、零开销调度、确定性推理），参考价值高；② SWA / 确定性这类优化带真实代码与等价性验证脚本，不是 PPT。
- **缺点**：① 只支持单一模型与特定硬件，通用性差；② KV 存储、P/D 分离、LB 未开源，无法端到端复现其生产系统。
