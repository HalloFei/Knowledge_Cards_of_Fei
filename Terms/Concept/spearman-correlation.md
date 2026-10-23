{
  "term": "Spearman相关性",
  "full_form": "Spearman's Rank Correlation Coefficient",
  "folder": "Concept",
  "definition": "基于数据秩次的非参数统计方法，衡量两变量间的单调相关关系",
  "tags": ["statistics", "correlation", "nonparametric", "ranking"],
  "storage_path": "Terms/Concept/spearman-correlation.md"
}

## 📖 详细定义
Spearman秩相关系数（ρ）通过将原始数据转换为秩次后计算相关性，值域为[-1, 1]。与Pearson不同，它不要求数据服从正态分布或呈线性关系，只要求单调关系。核心思想：将每个变量的数据从小到大排序赋秩，再对秩次计算Pearson相关系数。

## 实际应用
在生物信息学（基因表达相关性网络）、金融风控（非正态收益分布）、推荐系统（排序质量评估）、医学统计（非正态临床指标关联分析）等领域广泛使用。作为特征选择的常用工具，尤其在数据含异常值或偏态分布时优先选用。目前仍是数据分析与机器学习预处理的主流方法，无直接替代方案，但常与Kendall's τ互补使用。

## 🔬 前沿研究（2 项）
- `[2024] Large dimensional Spearman's rank correlation matrices`：建立高维Spearman相关矩阵的线性谱统计中心极限定理，提出三种新的高维独立性检验统计量。
- `[2025] Order Statistics Correlation Coefficient`：提出基于次序统计量的新型相关系数，综合了Pearson线性系数与Spearman/Kendall秩系数的优势，在生物信号分析中性能更优。

## 🔗 相关概念（3 个）
- `Pearson相关系数`：易混淆。Pearson衡量线性关系且要求正态分布，Spearman衡量单调关系且为非参数方法；数据异常时Spearman更稳健。
- `Kendall's τ`：互补。同为秩相关系数，τ基于一致对/不一致对计算，对小样本和含结数据更准确，但计算复杂度更高。
- `互信息（Mutual Information）`：父级概念。可捕捉任意类型的依赖关系（含非单调），但需估计概率分布；Spearman仅限单调关系但计算简便。

## ⚖️ 优缺点
- **优点**：①非参数、不要求正态性，对异常值鲁棒；②可检测任意单调关系（线性或非线性皆可）
- **缺点**：①仅能检测单调关系，对非单调依赖（如U型）不敏感；②存在结（tied ranks）时精确度下降，需修正公式