{
  "term": "MCP",
  "full_form": "Model Context Protocol",
  "folder": "Tool",
  "definition": "连接AI模型与外部工具和数据源的开放协议标准",
  "tags": ["protocol", "llm", "tool-use"],
  "storage_path": "KB/Tool/model-context-protocol.md"
}

📖 详细定义
MCP 是由 Anthropic 于 2024 年发布的开放协议，旨在标准化 AI 模型与外部工具、数据源之间的通信方式。采用客户端-服务器架构，MCP Server 暴露工具（Tools）、资源（Resources）和提示（Prompts），MCP Client（如 AI 助手）通过统一协议调用，实现"一次集成，多处复用"。
🔬 前沿研究
[2024] MCP Specification v1.0：Anthropic 发布 MCP 首版规范，定义 JSON-RPC 通信与工具发现机制，关键词：MCP specification。
[2025] MCP Ecosystem Growth：主流 AI 开发工具（Cursor、Claude Desktop 等）陆续原生支持 MCP，生态快速扩展，关键词：MCP ecosystem。
🔗 相关概念
Function Calling：互补——Function Calling 是模型调用函数的接口，MCP 是跨平台标准化的工具发现与调用协议层。
Tool Use：父概念——MCP 是 Tool Use 的一种标准化实现方案。
API：互补——API 是服务端定义的接口，MCP 在其上增加统一发现、描述与调用规范，降低集成碎片化。
⚖️ 优缺点
优点：1. 统一协议消除重复集成，工具一次封装即可被多客户端使用；2. 开放标准，社区驱动，避免厂商锁定。
缺点：1. 协议仍在早期阶段，规范与生态尚未完全成熟；2. 增加一层抽象，简单场景下可能过度设计。
