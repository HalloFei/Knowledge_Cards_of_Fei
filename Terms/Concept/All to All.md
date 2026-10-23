{
  "term": "All to All",
  "full_form": "All to All",
  "folder": "Concept",
  "definition": "系统中所有节点与所有节点直接互联的拓扑结构",
  "tags": ["network", "topology", "communication"],
  "storage_path": "Concept/All to All.md"
}

## 📖 详细定义
All to All（全互联）是一种网络或系统拓扑结构，指系统中每个节点都与其他所有节点建立直接连接。在并行计算中，All to All通信指所有处理器同时向所有其他处理器发送数据，常用于矩阵转置、FFT等操作。

## 实际应用
在高性能计算（HPC）和分布式深度学习中广泛应用。GPU集群训练大模型时，All to All通信是关键瓶颈之一。主流实现包括MPI_Alltoall、NCCL的all-to-all原语。随着模型并行和流水线并行的发展，All to All通信模式成为分布式训练性能优化的重要方向。竞争方案包括Ring All-Reduce（减少通信量）等优化策略。

## 🔬 前沿研究
- `[2024] 高带宽互连技术`：NVLink 4.0等技术提升All to All通信带宽，支持更大规模GPU集群。
- `[2023] 通信优化算法`：研究拓扑感知的All to All调度策略，降低网络拥塞和延迟。

## 🔗 相关概念
- `All Reduce`：所有节点规约到同一结果，与All to All相比通信模式更简单。
- `点对点通信`：节点间一对一通信，是All to All的基础单元。
- `通信拓扑`：All to All是其中一种模式，其他包括Ring、Tree等。