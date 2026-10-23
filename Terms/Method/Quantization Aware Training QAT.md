{
  "term": "QTA",
  "full_form": "Quantization Aware Training (可能歧义: QTA也可是Quality Time Assessment等其他缩写)",
  "folder": "Method",
  "definition": "训练时引入量化误差以优化低精度模型性能的方法",
  "tags": ["quantization", "model-compression", "deep-learning"],
  "storage_path": "Terms/Method/Quantization Aware Training QAT.md"
}
```

## 📖 详细定义
量化感知训练在模型训练过程中模拟量化误差,使模型学习适应低精度表示,从而在推理时使用INT8等低精度格式仍保持高准确率。与训练后量化(PTQ)不同,QAT需要在训练阶段就考虑量化影响。

## 实际应用
QAT是当前工业界部署高性能低精度模型的主流方案之一,广泛应用于移动端推理、边缘计算和云端推理加速。与PTQ相比,QAT通常能获得更高的精度,但需要额外的训练成本和时间。主流框架如TensorRT、PyTorch、TensorFlow都支持QAT。竞争方案包括PTQ(无训练成本)和混合精度量化。

## 🔬 前沿研究
- `[2024] QAT with Learned Step Size Quantization`: 自适应学习量化步长,在保持精度的同时实现更激进的量化
- `[2023] Binary Neural Networks via QAT`: 通过量化感知训练实现二值化网络,大幅压缩模型尺寸

## 🔗 相关概念
- `PTQ (Post-Training Quantization)`: 父概念,无需训练的量化方法,速度快但精度损失较大
- `INT8 Quantization`: 子概念,将FP32模型量化为8位整数表示,是QAT的常见目标精度
- `Mixed Precision Training`: 互补概念,训练时混合使用FP16和FP32,QAT可在此基础上进一步量化

## ⚖️ 优缺点
- **优点**: 比PTQ精度更高,能有效补偿量化误差,适合对精度要求高的场景
- **缺点**: 需要额外的训练时间和计算资源,实现复杂度高于PTQ