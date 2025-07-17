● server.py 是一个 FastAPI 代理服务器，核心逻辑分为以下几个部分：

  1. 配置与初始化

  - 环境变量加载：从 .env 文件读取 API 密钥和配置
  - 日志配置：定制化的日志系统，过滤 LiteLLM 的冗余日志
  - 全局变量：定义支持的模型列表和映射规则

  2. 模型映射系统

  - 智能映射：将 Claude 模型名（haiku/sonnet）映射到实际的后端模型
  - 提供商选择：基于 PREFERRED_PROVIDER 决定使用 OpenAI、Google 还是 Moonshot
  - 前缀自动添加：为模型名自动添加 openai/、gemini/、moonshot/ 前缀

  3. 请求转换流程

  Anthropic → LiteLLM

  - 消息格式转换：将 Anthropic 的消息结构转换为 OpenAI 格式
  - 系统消息处理：将 system prompt 转换为 OpenAI 的系统消息
  - 工具转换：将 Anthropic 的工具定义转换为 OpenAI 的 function calling 格式
  - 内容块处理：处理文本、图片、工具调用等不同类型的内容块
  - Gemini 特殊处理：清理 Gemini 不支持的 JSON schema 字段

  LiteLLM → Anthropic

  - 响应结构转换：将 OpenAI 响应转换回 Anthropic 格式
  - 工具调用处理：将 function calls 转换为 Anthropic 的 tool_use 块
  - 流式响应：处理 SSE 格式的流式响应
  - 错误处理：优雅的错误处理和回退机制

  4. API 端点

  - /v1/messages - 主要聊天完成端点
  - /v1/messages/count_tokens - 令牌计数端点

  5. 特殊处理逻辑

  - 模型能力检测：根据模型类型决定如何处理工具调用
  - 流式事件模拟：模拟 Anthropic 的流式事件格式
  - 内容清理：处理 OpenAI 不支持的复杂内容结构
  - 超时与重试：内置的错误处理和日志记录

  6. 流式响应处理

  - 事件类型：处理 message_start、content_block_start、content_block_delta 等事件
  - 工具调用流式化：支持工具参数的渐进式传输
  - 连接保持：处理 ping 事件保持连接活跃

  这个代理的核心价值在于让任何兼容 Anthropic API 的客户端（如 Claude Code）都能透明地使用 OpenAI、Gemini 或
  Moonshot 的模型，无需修改客户端代码。

> 他有多少个接口

● server.py 中定义了 3 个 HTTP 接口：

  1. POST /v1/messages - 主要的聊天完成接口（Anthropic 兼容）
  2. POST /v1/messages/count_tokens - 令牌计数接口
  3. GET / - 根路径，返回简单的健康检查消息

  所有接口都在 FastAPI 应用中定义，其中 /v1/messages 是最核心的接口，支持流式和非流式响应。