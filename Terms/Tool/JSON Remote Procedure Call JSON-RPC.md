{
  "term": "JSON-RPC",
  "full_form": "JSON Remote Procedure Call",
  "folder": "Tool",
  "definition": "基于JSON的轻量级远程过程调用协议",
  "tags": ["protocol", "rpc", "api", "json"],
  "storage_path": "KB/Tool/json-rpc.md"
}

📖 详细定义
JSON-RPC 是一种无状态、轻量级的远程过程调用协议，使用 JSON 作为数据格式。客户端发送包含方法名和参数的请求对象，服务端返回结果或错误对象。当前广泛使用的是 2.0 版本，支持批量调用和通知（无需响应的单向调用）。

🔬 前沿研究
[2010] JSON-RPC 2.0 Specification：正式发布 2.0 规范，引入命名参数、批量请求和结构化错误码，关键词：JSON-RPC 2.0 spec。
[2024] MCP adopts JSON-RPC：Anthropic 的 Model Context Protocol 选用 JSON-RPC 2.0 作为底层通信协议，推动其在 AI 工具生态中的新应用，关键词：MCP JSON-RPC transport。

🔗 相关概念
gRPC：易混淆——gRPC 使用 Protocol Buffers 和 HTTP/2，性能更高但更重；JSON-RPC 更轻量、易调试。
REST API：互补/对比——REST 面向资源、用 HTTP 动词语义；JSON-RPC 面向动作、统一用 POST，结构更简单。
XML-RPC：父概念——JSON-RPC 的前身，用 XML 编码，JSON-RPC 以 JSON 替代 XML 实现更轻量的同类功能。

⚖️ 优缺点
优点：1. 极简协议，规范仅几页，实现与集成成本低；2. 传输层无关，可运行在 HTTP、WebSocket、stdio 等之上。
缺点：1. 缺乏内建的服务发现与接口描述机制（无类似 OpenAPI 的标准）；2. 无类型约束，参数校验需额外实现。