# 一枚 token 进卡之后发生了什么:LLM 推理时 GPU 的完整参与方式

**副标题**: GPU 参与推理概念笔记——forward pass / 权重常驻 HBM / prefill vs decode / 算术强度与屋顶线 / KV cache / kernel 与 CUDA Graph / vLLM 的调度角色

**日期**: 2026-08-18

**可视化**: 图示化版本见同目录 [`gpu-llm-inference.html`](gpu-llm-inference.html)(token 进卡之旅、prefill vs decode 对比、一步 decode 的时间账、batch 杠杆、四类优化定位图,单文件离线可开)

**前置**: 本文默认已读 [`20260813T183934--gpu-memory-hierarchy__concept.md`](20260813T183934--gpu-memory-hierarchy__concept.md)(GPU 内存金字塔:register / SRAM / HBM / CPU RAM 谁在哪层、谁快谁大)

**关联论文笔记**: [`20260813T160000--paper-paged-attention-vllm__paper.md`](20260813T160000--paper-paged-attention-vllm__paper.md)

**来源**: 概念整理笔记(非单篇论文;背景取自 vLLM/PagedAttention 论文、CUDA 编程模型与 A100 公开规格)

---

## 速读

接上次的内存金字塔,这次把镜头从"卡里的存储长什么样"推近到"一次真实的 LLM 推理,数据在这座塔上怎么流"。要害只有一句话:**GPU 根本不认识 Transformer——它每一步只是被推理框架(vLLM 之类)喂进一小段数据和一个 kernel(一段在 SM 上跑的函数),而 LLM 推理的特殊性在于"权重几十 GB 常驻 HBM,每生成一个 token 都要把它们从 HBM 完整读一遍,却只做很少的计算"。** 这直接把推理切成两种完全不同的形态:prefill(一口气并行处理整段 prompt,大矩阵乘,卡的是**算力**)和 decode(每步只算一个新 token,矩阵退化成向量,卡的是**显存带宽**)。你感受到的"第一个字慢、后面每个字快但稳定地慢"正是这两种形态;而"加大 batch""KV cache 分页""CUDA Graph"这些 vLLM 的招牌动作,全都只是在这条流水线的不同环节上,把 GPU 的某一层资源喂得更饱而已。

## 它到底在解决什么

三个具体困惑,贯穿全文:

1. 同一个模型、同一张卡,为什么"吐出第一个 token"明显慢(TTFT),之后每个 token 都快一些但速度稳定(TPOT)?这两个数像是两种完全不同的工作。
2. 都说 decode 阶段"受限于显存带宽而不是算力"——可 GPU 明明有几百 TFLOPS 的算力,它怎么会闲着?瓶颈到底卡在哪一层的资源上?
3. GPU 自己显然不懂"Transformer 推理"这件事。从输入一段 prompt 到吐出 token,GPU 究竟是被谁、按什么粒度驱动的?vLLM 在其中扮演什么角色?

要解开后两个,得先承认:**GPU 的全部聪明只在"把一个 kernel 跑得飞快",它对 LLM 一无所知;"理解并编排一次推理"是框架在 CPU 上做的,GPU 全程被动接活。** 而第一个困惑的钥匙,是把"一次前向"在 GPU 上的真实数据流量算出来——一旦算账,三种困惑其实是同一笔账。

## 它真正看见了什么

### GPU 不认识 LLM,它只认识 kernel

先建立最小图景。你给 vLLM 发一段 prompt,框架在 CPU 上做的事是:把 token id 组成张量、准备好 KV cache 的地址表,然后把模型的一层层的算子翻译成**一串 CUDA kernel(GPU 上执行的函数)**,依次发射(launch)到 GPU 的 stream 上异步执行。GPU 收到的从来不是"跑一个 Transformer",而是"跑这个 GEMM kernel"、"跑那个 attention kernel"——每次一小段。

一次前向(forward pass)的 kernel 串大致是:

```text
token ids ─> embedding 查表 ─> L 层循环 ──────────────┐
   每层: QKV 投影(GEMM) -> attention kernel           │
         -> 输出投影(GEMM) -> MLP(GEMM x2)            │
         -> (+残差/layernorm 等小算子)                 │
   <──────────────────────────────────────────────────┘
        ─> lm_head(GEMM, 词表 x 维度) ─> logits ─> 采样 ─> 新 token
```

关键不在 kernel 名字,而在**数据的量级差**:权重(几十 GB)常驻 HBM;激活值和 logits(MB 级)只在 HBM 短暂停留,进 kernel 时再搬进片上 SRAM 计算。所以一次前向的真实动作是:**SM 阵列轮流把每一份权重从 HBM 经 L2 读进来,乘完就丢,换下一份。** 每生成一个 token,这套搬运就要完整走一遍——这是后面所有算账的底座。

### prefill 与 decode:同一个模型,两种完全不同的负载

上面那条 kernel 串在两个阶段形态完全不同。**prefill(预填充)** 处理你发来的整段 prompt:N 个 token **并行**算,每个 GEMM 的形状是"大矩阵 × 大矩阵",SM 阵列吃得满,计算量随 N 线性涨、而权重只读一遍。**decode(解码)** 每步只算**一个**新 token:同样的权重、同样的层,但 GEMM 退化成"矩阵 × 向量"(GEMV)——还是要把全部权重读一遍,却只做 1/N 的计算。

```text
              读权重(HBM 流量)   计算量(FLOPs)
prefill(N=1k)   ~14 GB(一遍)      ~N 倍:大算大乘  -> 卡算力
decode(每步)    ~14 GB(每步!)    ~1 倍:大读小乘  -> 卡带宽
```

这就是第一个困惑的答案:**TTFT 慢在 prefill(一段长 prompt 的完整前向,但吞吐高),TPOT 慢在 decode(每步都重读全部权重,但计算稀薄)。** 它们不是快慢不同的同一件事,而是瓶颈资源不同的两种负载。

### 用算术强度算这笔账:为什么 decode 卡带宽、卡在哪层

判断"卡算力还是卡带宽"有一把标准尺子:**算术强度(arithmetic intensity)= 每读/写 1 byte 数据要做多少次 FLOP**,对着屋顶线(roofline)模型一比就知道瓶颈。拿 7B 模型 FP16 在 A100 上算 decode 的账:

```text
权重 14 GB, 每步读一遍      A100 HBM 带宽 ~2 TB/s   -> 搬运至少 ~7 ms
每步计算 ~14 GFLOP          A100 算力 ~312 TFLOPS   -> 计算只需 ~0.05 ms
瓶颈 = max(7, 0.05) = 带宽, 差着 100 倍以上
```

也就是说 decode 时**算力只能用到不足 1%**,GPU 大部分晶体管在等数据从 HBM 流过——这正是内存金字塔里"片上快而小、片下大而慢"的张力,在推理负载上的直接显形:权重太大放不进片上,只能住在 HBM,于是每一步都要穿过那道最窄的门。prefill 恰好相反:一次读权重要做 N 倍计算,算术强度随 N 上升,瓶颈翻转到算力一侧。

**改变一个条件就能看到杠杆在哪:batch。** decode 时 B 个请求拼成一个 batch,权重大部分层只需读一遍就能同时服务 B 个请求(HBM 流量几乎不变),计算量翻 B 倍——算术强度被凭空抬起来,batch 越大越滑向算力受限,GPU 利用率越高。vLLM 的 **continuous batching**(步与步之间动态插入/移出请求)本质上就是不断把这个 B 维持在尽可能大,让每次读权重"物尽其用"。

### KV cache:decode 为什么必须带一套越滚越大的行李

attention 那一步有个特殊要求:算第 t 个 token 的注意力,需要**全部前 t-1 个 token 的 K 和 V**。没有缓存的话每步都要重算整段历史——代价 O(t²)。所以框架把每层的 K/V 存进 HBM,即 **KV cache**:每步只算新 token 的 Q/K/V,历史直接查表。它是 decode 高吞吐的命根子,也带来两笔代价:它随序列长度线性膨胀,**决定了一张卡能同时接待多少请求**(batch 上限常常不是算力,而是"KV cache 塞不下了");它的读排布方式直接决定 attention kernel 的带宽效率——PagedAttention 论文整篇就在解决"KV cache 在 HBM 内部怎么分页不浪费",这正是金字塔笔记里说的"在 HBM 这一层内部省容量"。

### kernel launch 与 CUDA Graph:瓶颈还有一段在 CPU

decode 的 kernel 串还有一个非 GPU 的隐患:每步几十层 × 若干小算子 = 几百个 kernel,**每个都由 CPU 逐个发射**,每次发射有几微秒开销;而单个 kernel 本身可能只跑几十微秒。batch 小时,CPU 发射速度赶不上 GPU 消化速度,GPU 就在两次 kernel 之间空转。**CUDA Graph** 的解法是:把一整步 decode 的整条 kernel 串预先录制一遍,之后每步只发射"重放整张图"这一次调用,把几百次 launch 压成一次。vLLM 对 decode 路径默认启用它,原因就在这——这也回答了第三个困惑的最后一半:框架不仅决定"跑什么",还要保证"喂得够快"。

### 合起来:vLLM 一步 decode 的完整循环

```text
CPU(vLLM 调度器)                     GPU
┌────────────────────┐    PCIe      ┌───────────────────────────────┐
│ 组 batch / 换入换出  │ ──────────> │ HBM: 权重(常驻) + KV cache(膨胀) │
│ 查 block 表, 发 kernel│  发射 kernel │      + 本步 token ids(几个字节) │
│ (decode 走 CUDA Graph)│  流水线发射  │  │ 读                          │
└────────────────────┘              │  ▼                            │
        ▲ 下一轮把新 token 塞回 batch  │ SM 阵列: GEMV / attention / MLP │
        └──────────────────────────  │  ▼ 写回                        │
                                     │ HBM: logits ─> 采样 ─> 新 token │
                                     └───────────────────────────────┘
```

GPU 在这个循环里始终只做三件事:**收 kernel、按塔搬运数据、交回 logits**;什么时候收、收多大批,全由 CPU 上的调度器决定。所谓"GPU 参与 LLM 推理",就是这台被动但极快的机器,被框架按 prefill/decode 两种形态、以 kernel 为粒度持续喂活的过程。

**区分一个邻近情况**:prefill 每步同样要把全部权重读一遍,为什么没人说它带宽受限?因为算术强度不同——prefill 读一遍权重要做 N 份 token 的计算,带宽开销被 N 摊薄;decode 读一遍只服务 1 份。同一条 kernel 串,负载形状变了,瓶颈楼层就换了,这正是"先问瓶颈在哪层"这把尺子的价值。

## 我能带走什么

> 以后看到任何 LLM 推理系统的优化(vLLM、SGLang、TensorRT-LLM……),先别记技巧,先定位:**它优化的是这条流水线的哪一环?**

把常见动作挂回流水线,一共四环:(a) **让 HBM 里的大块头更省**——量化压缩权重、PagedAttention 分页 KV cache(省容量);(b) **让每次读带宽更值**——continuous batching 抬算术强度、prefix cache 复用已算好的 KV(省流量);(c) **减少 HBM↔片上 SRAM 的来回**——FlashAttention、算子融合(省搬运);(d) **消灭 GPU 之外的空转**——CUDA Graph、调度与计算重叠(省发射/等待)。识别信号沿用金字塔笔记那条:听它反复念叨的是"容量/占用"、"带宽/复用"、"搬运/融合"还是"launch/调度",就知道它在四环中的哪一环发力——而四环加起来,就是 GPU 参与 LLM 推理的全部戏份。