```json
{
  "term": "Batch Invariance",
  "full_form": "Batch Invariance",
  "folder": "Concept",
  "definition": "同一输入在任意批组成下得到逐位相同输出的性质",
  "tags": ["determinism", "numerics", "llm-inference", "reproducibility"],
  "aliases": ["批不变性", "批次不变性", "batch-invariant inference"],
  "storage_path": "Terms/Concept/Batch Invariance.md"
}
```

## 📖 详细定义

对任意两条请求，单独跑和混在任意 batch 里跑，logits / logprobs / 输出 token **逐位相同**。注意它比「结果接近」强得多：要求 bitwise，不是 allclose。

根因不是随机性，是**归约顺序**：

- GEMM 的 split-K：K 维切给多组 CTA 并行累加，切分数由启发式按 shape 决定 → 随 batch 变；
- attention 的 KV 切分与 tile 调度、MoE 的多路归约、RMSNorm 的 warp 归约树、sampler 的 LSE / 直方图 tile，同理。

浮点加法不满足结合律，`(a+b)+c ≠ a+(b+c)`，末位就漂了。

三条实现路线：① **固定划分**（K 不切、tile 大小与 batch 无关）；② **换精确算术**（定点整数累加，atomic 顺序无关）；③ **禁用随形状变体的库路径**（cuBLASLt heuristics、依赖 batch 形状的融合 kernel、多路归约）。

常见误区：以为固定 seed 就确定了。seed 只固定随机数，不固定浮点归约。

## 实际应用

- 三个主流框架都有了开关：vLLM `VLLM_BATCH_INVARIANT=1`（beta）、SGLang `--enable-deterministic-inference`（限 flashinfer / fa3 / triton 后端）、DeepInfer `DEEPINFER_BATCH_INVARIANT=1`（仅 NVIDIA）。
- 刚需场景：RL rollout 可复现（GRPO 同一 prompt 多采样的多样性与可复现性要同时成立）、评测分数与分片方式无关、算子/模型数值回归（可以用 KL=0 做硬断言）、框架 debug。
- 代价是实打实的：关 split-K、关融合 kernel、关多路归约，吞吐下降；DeepInfer 的评测脚本 help 里直接标了 "(slower)"。
- 竞争关系：它是「正确性/可复现性」维度的能力，和「吞吐」维度天然冲突；生产上通常只在评测、RL、回归链路开，在线服务不开。

## 🔬 前沿研究（3 项）

- `[2025] Defeating Nondeterminism in LLM Inference`（Thinking Machines Lab）：系统论证「batch invariance 是内核级不变量」，是 vLLM / SGLang 相关实现的共同源头（检索关键词：defeating nondeterminism batch invariant kernels）。
- `[2025] vLLM Batch Invariance 文档`：工程化清单——TP all-reduce 固定归约序、LoRA 分块归约、breakable CUDA graph、禁用 FlashInfer/AITER/QuickReduce 归约（检索关键词：vllm batch invariance）。
- `[2026] DeepInfer`：把该性质扩到 MoE-EP（DeepEP 多路归约关闭）、投机验证、多模态编码器，并作为确定性采样的前置条件（检索关键词：DeepInfer batch invariant）。

## 🔗 相关概念（3 个）

- `Deterministic Sampling`：下游——前向 bitwise 一致只是必要条件，采样器还要无状态。
- `Split-K GEMM`：头号元凶——K 切分数随 batch 变，是最主要的不确定性来源。
- `Reduction Order`：更上位的规律——LayerNorm / MoE / sampler 里的所有浮点归约都受同一条规律支配。

## ⚖️ 优缺点

- **优点**：① 给「同输入同输出」提供可验证的强契约，RL、评测、数值回归都能直接用；② 把「数值 bug」和「batch 相关的抖动」分离开，debug 成本大幅下降。
- **缺点**：① 必须牺牲 split-K、融合 kernel 等吞吐优化；② 覆盖不完整时会出现「部分算子不变、部分算子变」，比完全不确定更难排查。
