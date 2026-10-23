{
  "term": "F1 Score",
  "full_form": "F1 Score",
  "folder": "Tool",
  "definition": "精确率与召回率的调和平均数，综合衡量分类性能",
  "tags": ["evaluation", "classification", "metrics"],
  "storage_path": "KB/Tool/f1-score.md"
}

📖 详细定义
F1 Score 是 Precision 和 Recall 的调和平均值（2·P·R/(P+R)），取值范围 [0,1]，在正负样本不平衡时比准确率更能反映模型对正类的识别能力。
🔬 前沿研究（2 项）
[2020] Micro/Macro/Weighted F1 in Multi-label Settings：系统比较了多标签分类中不同 F1 聚合策略对模型评估的影响。检索关键词：macro micro weighted F1 multi-label。
[2023] BERTScore vs F1 for Generation Tasks：探讨在文本生成任务中 token-level F1 与语义匹配指标的互补性。检索关键词：BERTScore token F1 generation evaluation。
🔗 相关概念（3 个）
Precision：F1 的组成部分之一，衡量预测为正的样本中真正为正的比例。
Recall：F1 的另一组成部分，衡量所有真正正样本中被正确预测的比例。
AUC-ROC：互补指标；AUC 评估所有阈值下的整体排序能力，F1 依赖于固定阈值。
⚖️ 优缺点
优点：① 对不平衡数据集比 Accuracy 更有意义；② 同时兼顾查准与查全，单一数值便于比较。
缺点：① 默认等权重对待 Precision 和 Recall，不适用于成本不对称场景；② 仅针对二分类正类，多类需额外聚合策略。


