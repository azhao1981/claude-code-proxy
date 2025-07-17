# Anthropic API 代理：用于 Gemini 和 OpenAI 模型的桥梁 🔄

**让 Anthropic 客户端（如 Claude Code）使用 Gemini 或 OpenAI 作为后端。** 🤝

这是一个代理服务器，通过 LiteLLM 让 Anthropic 客户端能够使用 Gemini 或 OpenAI 模型。🌉

## 快速开始 ⚡

### 前置要求

- OpenAI API 密钥 🔑
- Google AI Studio（Gemini）API 密钥（如果使用 Google 提供商）🔑
- 已安装 [uv](https://github.com/astral-sh/uv)

### 设置 🛠️

1. **克隆仓库**：
   ```bash
   git clone https://github.com/1rgs/claude-code-openai.git
   cd claude-code-openai
   ```

2. **安装 uv**（如果尚未安装）：
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```
   *（`uv` 会根据 `pyproject.toml` 自动处理依赖）*

3. **配置环境变量**：
   复制示例环境文件：
   ```bash
   cp .env.example .env
   ```
   编辑 `.env` 并填写 API 密钥和模型配置：

   * `ANTHROPIC_API_KEY`：（可选）仅当代理到 Anthropic 模型时需要
   * `OPENAI_API_KEY`：您的 OpenAI API 密钥（默认使用 OpenAI 时需要）
   * `GEMINI_API_KEY`：您的 Google AI Studio（Gemini）API 密钥（如果 `PREFERRED_PROVIDER=google` 则需要）
   * `PREFERRED_PROVIDER`（可选）：设为 `openai`（默认）或 `google`，决定 `haiku`/`sonnet` 的主后端
   * `BIG_MODEL`（可选）：映射 `sonnet` 请求的模型，默认为 `gpt-4.1`（如果 `PREFERRED_PROVIDER=openai`）或 `gemini-2.5-pro-preview-03-25`
   * `SMALL_MODEL`（可选）：映射 `haiku` 请求的模型，默认为 `gpt-4.1-mini`（如果 `PREFERRED_PROVIDER=openai`）或 `gemini-2.0-flash`

   **映射逻辑：**
   - 如果 `PREFERRED_PROVIDER=openai`（默认），`haiku`/`sonnet` 映射到带 `openai/` 前缀的 `SMALL_MODEL`/`BIG_MODEL`
   - 如果 `PREFERRED_PROVIDER=google`，`haiku`/`sonnet` 映射到带 `gemini/` 前缀的 `SMALL_MODEL`/`BIG_MODEL`（如果这些模型在已知 `GEMINI_MODELS` 列表中，否则回退到 OpenAI 映射）

4. **运行服务器**：
   ```bash
   uv run uvicorn server:app --host 0.0.0.0 --port 8082 --reload
   ```
   *（`--reload` 可选，用于开发）*

### 与 Claude Code 一起使用 🎮

1. **安装 Claude Code**（如果尚未安装）：
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```

2. **连接到您的代理**：
   ```bash
   ANTHROPIC_BASE_URL=http://localhost:8082 claude
   ```

3. **完成！** 您的 Claude Code 客户端现在将通过代理使用配置的后端模型（默认为 Gemini）。🎯

## 模型映射 🗺️

代理根据配置的模型自动将 Claude 模型映射到 OpenAI 或 Gemini 模型：

| Claude 模型 | 默认映射 | 当 BIG_MODEL/SMALL_MODEL 为 Gemini 模型时 |
|-------------|----------|--------------------------------------|
| haiku       | openai/gpt-4o-mini | gemini/[模型名称] |
| sonnet      | openai/gpt-4o | gemini/[模型名称] |

### 支持的模型

#### OpenAI 模型
以下 OpenAI 模型受支持，自动处理 `openai/` 前缀：
- o3-mini
- o1
- o1-mini
- o1-pro
- gpt-4.5-preview
- gpt-4o
- gpt-4o-audio-preview
- chatgpt-4o-latest
- gpt-4o-mini
- gpt-4o-mini-audio-preview
- gpt-4.1
- gpt-4.1-mini

#### Gemini 模型
以下 Gemini 模型受支持，自动处理 `gemini/` 前缀：
- gemini-2.5-pro-preview-03-25
- gemini-2.0-flash

### 模型前缀处理
代理自动为模型名称添加适当前缀：
- OpenAI 模型获得 `openai/` 前缀
- Gemini 模型获得 `gemini/` 前缀
- BIG_MODEL 和 SMALL_MODEL 将根据它们在 OpenAI 或 Gemini 模型列表中的位置获得适当前缀

例如：
- `gpt-4o` 变成 `openai/gpt-4o`
- `gemini-2.5-pro-preview-03-25` 变成 `gemini/gemini-2.5-pro-preview-03-25`
- 当 BIG_MODEL 设为 Gemini 模型时，Claude Sonnet 将映射到 `gemini/[模型名称]`

### 自定义模型映射

使用 `.env` 文件中的环境变量或直接控制映射：

**示例 1：默认（使用 OpenAI）**
无需在 `.env` 中更改，除了 API 密钥，或确保：
```dotenv
OPENAI_API_KEY="您的-openai-密钥"
GEMINI_API_KEY="您的-google-密钥" # 如果 PREFERRED_PROVIDER=google 则需要
# PREFERRED_PROVIDER="openai" # 可选，这是默认设置
# BIG_MODEL="gpt-4.1" # 可选，这是默认设置
# SMALL_MODEL="gpt-4.1-mini" # 可选，这是默认设置
```

**示例 2：优先使用 Google**
```dotenv
GEMINI_API_KEY="您的-google-密钥"
OPENAI_API_KEY="您的-openai-密钥" # 需要回退
PREFERRED_PROVIDER="google"
# BIG_MODEL="gemini-2.5-pro-preview-03-25" # Google 偏好时的可选默认设置
# SMALL_MODEL="gemini-2.0-flash" # Google 偏好时的可选默认设置
```

**示例 3：使用特定 OpenAI 模型**
```dotenv
OPENAI_API_KEY="您的-openai-密钥"
GEMINI_API_KEY="您的-google-密钥"
PREFERRED_PROVIDER="openai"
BIG_MODEL="gpt-4o" # 示例特定模型
SMALL_MODEL="gpt-4o-mini" # 示例特定模型
```

## 工作原理 🧩

此代理通过以下方式工作：

1. **接收** Anthropic API 格式的请求 📥
2. **转换** 通过 LiteLLM 将请求转换为 OpenAI 格式 🔄
3. **发送** 转换后的请求到 OpenAI 📤
4. **转换** 将响应转换回 Anthropic 格式 🔄
5. **返回** 格式化后的响应给客户端 ✅

代理处理流式和非流式响应，与所有 Claude 客户端保持兼容性。🌊

## 贡献 🤝

欢迎贡献！请随时提交拉取请求。🎁