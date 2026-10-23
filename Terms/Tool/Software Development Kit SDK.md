{
  "term": "SDK",
  "full_form": "Software Development Kit",
  "folder": "Tool",
  "definition": "封装特定平台API与工具的开发套件，简化集成与开发",
  "tags": ["api", "development", "integration"],
  "storage_path": "KB/Tool/sdk.md"
}

📖 详细定义
SDK 是一组预封装的库、工具、文档和代码示例的集合，由平台或服务提供方发布，帮助开发者快速集成其能力而无需从零实现底层协议。通常包含 API 绑定、编译/调试工具及运行时依赖。
🔬 前沿研究
[2024] LLM Provider SDKs：OpenAI、Anthropic 等推出官方 SDK，标准化模型调用与流式响应处理，关键词：LLM SDK。
[2025] SDK Generation from OpenAPI Specs：基于 API 规范自动生成多语言 SDK 的工具链日趋成熟，关键词：auto SDK generation。
🔁 相关概念
API：互补——API 定义接口规范，SDK 是对 API 的封装与便捷调用层，SDK 内部调用 API。
CLI (Command Line Interface)：互补——SDK 面向代码集成，CLI 面向命令行交互，同一平台常同时提供两者。
Framework：易混淆——Framework 提供架构与控制流（IoC），SDK 提供工具与封装，控制权方向不同。
⚖️ 优缺点
优点：1. 大幅降低集成复杂度，开箱即用；2. 官方维护，版本同步更新，减少兼容性风险。
缺点：1. 引入额外依赖，增加包体积与供应链风险；2. 绑定特定平台，迁移成本高，存在厂商锁定。
已为术语 SDK (Software Development Kit) 生成结构化归档卡片与精简学习笔记。SDK 是封装平台 API 与工具的开发套件，卡片建议存储路径为 KB/Tool/sdk.md。