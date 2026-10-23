```json
{
  "term": "BPE",
  "full_form": "Byte-Pair Encoding",
  "folder": "Method",
  "definition": "反复合并最高频相邻符号对，把文本切成子词单元的贪心分词算法",
  "tags": ["tokenization", "nlp", "llm", "subword"],
  "aliases": ["字节对编码"],
  "storage_path": "Terms/Method/Byte-Pair Encoding BPE.md"
}
```

## 📖 详细定义

BPE 最早不是 NLP 方法。1994 年 Philip Gage 用它做数据压缩：反复把出现频率最高的一对相邻字节替换成一个新符号，词表越压越小。

Sennrich 等人在 2016 年把它搬进机器翻译：从字符（或字节）出发，统计语料里相邻符号对的频次，把最高频的一对合并成新 token，重复 V 次，就得到一个大小为 V 的词表。这一步是**训练**。

**推理**时不再统计，只按学到的合并规则顺序贪心切分：从字符开始，遇到规则就合并，直到没有规则可用。

三个容易混的分支：

- **byte-level BPE**（GPT-2 起）：基础字母表不是 Unicode 字符，而是 256 个字节。好处是任何字符串都能表示，永远没有 `<UNK>`；代价是一个中文字符通常要占 2–4 个字节。
- **WordPiece**（BERT 一系）：合并判据不是频次，而是"合并后语言模型似然提升最大"。
- **Unigram**（SentencePiece，Kudo 2018）：方向相反，先给一个大词表，再按概率剪枝。

必须分清的一点：BPE 既不是按词切，也不是按字切，而是数据驱动的子词切分。词表大小是超参数，切分结果由语料统计决定——同一段文本，换一个语料训出来的 BPE，token 序列就不一样。

## 实际应用

- 几乎所有主流 LLM 都用它或它的变体：GPT 系列（tiktoken `cl100k_base` 约 10 万、`o200k_base` 约 20 万）、Llama 3（12.8 万）、Qwen（15 万）、Mistral 等。上一份 YOCO 笔记里那个 `tiktoken-cl100k_base` 就是这一族。
- 影响面比多数人以为的大：词表越大，同样文本的 token 越少、上下文更省，但 embedding 与输出 softmax 层越大；词表越小，序列越长，注意力和 KV cache 的成本跟着涨。分词器其实是一个"架构—成本"旋钮。
- 多语言与代码上的偏置是真实问题：英文常见词常常 1 token，一个汉字常 2–3 token；数字被按 3 位切（`1234` → `123`+`4`）会让算术更难；代码缩进与长标识符也常被切碎。
- 竞争方案：WordPiece（BERT 时代）、Unigram/SentencePiece（多语言更友好），以及 2024 年之后出现的 tokenizer-free 路线——直接在字节或动态补丁上建模，绕开固定词表。

## 🔬 前沿研究（3 项）

- `[2016] Neural Machine Translation of Rare Words with Subword Units (Sennrich et al., arXiv:1508.07909)`：把 BPE 引入 NMT，用子词单元解决未登录词，成为现代分词的事实起点（检索关键词：subword units BPE neural machine translation）。
- `[2019] GPT-2: Language Models are Unsupervised Multitask Learners（OpenAI 技术报告）`：改用 byte-level BPE，基础词表固定为 256 字节，彻底消除 `<UNK>`，此后 GPT 与 Llama 一系基本沿用（检索关键词：byte-level BPE GPT-2 tokenizer）。
- `[2024] Byte Latent Transformer: Patches Scale Better Than Tokens (Meta, arXiv:2412.09871)`：抛弃固定词表，按字节熵动态切"补丁"，同等算力下匹配或超过 BPE 分词器，正面质疑分词这一步本身（检索关键词：byte latent transformer tokenizer-free dynamic patching）。

## 🔗 相关概念（3 个）

- `Tokenization（分词）`：上位概念——BPE 是分词算法的一种，决定"文本怎样变成模型实际看到的符号序列"。
- `WordPiece / Unigram`：同层竞争方案——都是子词切分，差别在判据：BPE 看频次，WordPiece 看似然，Unigram 看概率。
- `Embedding 层与词表大小`：下游约束——词表大小直接决定 embedding 参数量、序列长度与 KV cache 成本，是模型设计中少数几个"一动就全动"的参数。

## ⚖️ 优缺点

- **优点**：① 没有 OOV，任意字符串都能表示；② 在"整词"和"单字"之间取得压缩率与泛化的平衡，词表大小可调。
- **缺点**：① 切分对语料分布敏感，多语言、代码、数字上存在系统性偏置；② 分词是不可微的前处理，切错会一路传下去，而且不同模型的分词器互不兼容，换模型就得重算全部 token 统计。
