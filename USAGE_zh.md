# Claw Code 使用说明

本指南覆盖 `rust/` 下当前的 Rust 工作区以及 `claw` CLI 二进制。如果你是第一次接触它，请把 doctor 健康检查作为你的第一次运行：先启动 `claw`，然后执行 `/doctor`。

## 快速启动健康检查

在使用 prompt、session 或自动化之前，请先运行：

```bash
cd rust
cargo build --workspace
./target/debug/claw
# REPL 内的第一条命令
/doctor
```

`/doctor` 是内置的安装与预检诊断命令。一旦你已经有了保存好的 session，就可以通过 `./target/debug/claw --resume latest /doctor` 再次运行它。

## 前置条件

- 安装带有 `cargo` 的 Rust 工具链
- 以下二选一：
  - 使用 `ANTHROPIC_API_KEY` 直接访问 API
  - 使用 `claw login` 进行基于 OAuth 的认证
- 可选：如果要连接代理或本地服务，可设置 `ANTHROPIC_BASE_URL`

## 安装 / 构建工作区

```bash
cd rust
cargo build --workspace
```

调试构建完成后，CLI 二进制位于 `rust/target/debug/claw`。建议把上面的 doctor 检查作为构建后的第一步。

## 快速开始

### 首次运行 doctor 检查

```bash
cd rust
./target/debug/claw
/doctor
```

### 交互式 REPL

```bash
cd rust
./target/debug/claw
```

### 一次性 prompt

```bash
cd rust
./target/debug/claw prompt "summarize this repository"
```

### 简写 prompt 模式

```bash
cd rust
./target/debug/claw "explain rust/crates/runtime/src/lib.rs"
```

### 供脚本使用的 JSON 输出

```bash
cd rust
./target/debug/claw --output-format json prompt "status"
```

## 模型与权限控制

```bash
cd rust
./target/debug/claw --model sonnet prompt "review this diff"
./target/debug/claw --permission-mode read-only prompt "summarize Cargo.toml"
./target/debug/claw --permission-mode workspace-write prompt "update README.md"
./target/debug/claw --allowedTools read,glob "inspect the runtime crate"
```

支持的权限模式：

- `read-only`
- `workspace-write`
- `danger-full-access`

CLI 当前支持的模型别名：

- `opus` → `claude-opus-4-6`
- `sonnet` → `claude-sonnet-4-6`
- `haiku` → `claude-haiku-4-5-20251213`

## 认证

### API key

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

### OAuth

```bash
cd rust
./target/debug/claw login
./target/debug/claw logout
```

### 环境变量应该怎么放

`claw` 接受两种 Anthropic 凭据环境变量，而且它们**不能混用**，因为 Anthropic 对不同凭据形态要求的 HTTP Header 不同。把值放错位置，是我们最常见的 401 来源。

| 凭据形态 | 环境变量 | HTTP Header | 常见来源 |
|---|---|---|---|
| `sk-ant-*` API key | `ANTHROPIC_API_KEY` | `x-api-key: sk-ant-...` | [console.anthropic.com](https://console.anthropic.com) |
| OAuth access token（不透明字符串） | `ANTHROPIC_AUTH_TOKEN` | `Authorization: Bearer ...` | `claw login` 或签发 Bearer token 的 Anthropic 兼容代理 |
| OpenRouter key（`sk-or-v1-*`） | `OPENAI_API_KEY` + `OPENAI_BASE_URL=https://openrouter.ai/api/v1` | `Authorization: Bearer ...` | [openrouter.ai/keys](https://openrouter.ai/keys) |

**为什么这很重要：** 如果你把一个 `sk-ant-*` key 填进 `ANTHROPIC_AUTH_TOKEN`，Anthropic API 会返回 `401 Invalid bearer token`，因为 `sk-ant-*` key 不能通过 Bearer Header 使用。修复方法只需要改一行环境变量：把 key 移到 `ANTHROPIC_API_KEY`。较新的 `claw` 构建会检测这种特定情况（401 + Bearer 槽位里是 `sk-ant-*`），并在错误信息里附带修复提示。

**如果你本来想用的是别的提供商：** 如果 `claw` 报告缺少 Anthropic 凭据，但你明明已经导出了 `OPENAI_API_KEY`、`XAI_API_KEY` 或 `DASHSCOPE_API_KEY`，那你大概率只是忘了给模型名加上提供商路由前缀。请使用 `--model openai/gpt-4.1-mini`（OpenAI 兼容 / OpenRouter / Ollama）、`--model grok`（xAI）或 `--model qwen-plus`（DashScope）；这样前缀路由器会优先选择正确后端，而不会被当前环境里其他凭据干扰。当前错误信息也会包含针对已检测到环境变量的提示。

## 本地模型

`claw` 可以通过 Anthropic 兼容端点或 OpenAI 兼容端点连接本地服务和提供商网关。Anthropic 兼容服务请使用 `ANTHROPIC_BASE_URL` 配合 `ANTHROPIC_AUTH_TOKEN`；OpenAI 兼容服务请使用 `OPENAI_BASE_URL` 配合 `OPENAI_API_KEY`。OAuth 仅适用于 Anthropic，因此当设置了 `OPENAI_BASE_URL` 时，应使用 API key 风格认证，而不是 `claw login`。

<a id="openai-compatible-endpoint"></a>

### Anthropic 兼容端点

```bash
export ANTHROPIC_BASE_URL="http://127.0.0.1:8080"
export ANTHROPIC_AUTH_TOKEN="local-dev-token"

cd rust
./target/debug/claw --model "claude-sonnet-4-6" prompt "reply with the word ready"
```

### OpenAI 兼容端点

```bash
export OPENAI_BASE_URL="http://127.0.0.1:8000/v1"
export OPENAI_API_KEY="local-dev-token"

cd rust
./target/debug/claw --model "qwen2.5-coder" prompt "reply with the word ready"
```

### Ollama

```bash
export OPENAI_BASE_URL="http://127.0.0.1:11434/v1"
unset OPENAI_API_KEY

cd rust
./target/debug/claw --model "llama3.2" prompt "summarize this repository in one sentence"
```

<a id="openrouter"></a>

### OpenRouter

```bash
export OPENAI_BASE_URL="https://openrouter.ai/api/v1"
export OPENAI_API_KEY="sk-or-v1-..."

cd rust
./target/debug/claw --model "openai/gpt-4.1-mini" prompt "summarize this repository in one sentence"
```

### 阿里云 DashScope（Qwen）

如果你要通过阿里原生 DashScope API 使用 Qwen 模型（它通常比 OpenRouter 具有更高的限流阈值）：

```bash
export DASHSCOPE_API_KEY="sk-..."

cd rust
./target/debug/claw --model "qwen/qwen-max" prompt "hello"
# 或者直接：
./target/debug/claw --model "qwen-plus" prompt "hello"
```

以 `qwen/` 或 `qwen-` 开头的模型名会被自动路由到 DashScope 兼容模式端点（`https://dashscope.aliyuncs.com/compatible-mode/v1`）。你**不需要**设置 `OPENAI_BASE_URL`，也不需要取消设置 `ANTHROPIC_API_KEY`，因为模型前缀会优先于环境凭据嗅探。

推理变体（`qwen-qwq-*`、`qwq-*`、`*-thinking`）会在请求真正发出前自动移除 `temperature` / `top_p` / `frequency_penalty` / `presence_penalty`（这些参数会被推理模型拒绝）。

## 支持的提供商与模型

`claw` 内置了三类提供商后端。它会根据模型名自动选择提供商；如果模型名无法明确判断，则退回到环境中已有的凭据。

### 提供商矩阵

| 提供商 | 协议 | 认证环境变量 | Base URL 环境变量 | 默认 Base URL |
|---|---|---|---|---|
| **Anthropic**（直连） | Anthropic Messages API | `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` 或 OAuth（`claw login`） | `ANTHROPIC_BASE_URL` | `https://api.anthropic.com` |
| **xAI** | OpenAI 兼容 | `XAI_API_KEY` | `XAI_BASE_URL` | `https://api.x.ai/v1` |
| **OpenAI-compatible** | OpenAI Chat Completions | `OPENAI_API_KEY` | `OPENAI_BASE_URL` | `https://api.openai.com/v1` |
| **DashScope**（阿里云） | OpenAI 兼容 | `DASHSCOPE_API_KEY` | `DASHSCOPE_BASE_URL` | `https://dashscope.aliyuncs.com/compatible-mode/v1` |

OpenAI-compatible 后端同时也是 **OpenRouter**、**Ollama** 以及任何实现了 OpenAI `/v1/chat/completions` 协议的服务的统一入口；只需要把 `OPENAI_BASE_URL` 指向对应服务即可。

**按模型名前缀路由：** 如果模型名以 `openai/`、`gpt-`、`qwen/` 或 `qwen-` 开头，提供商会根据前缀选择，而不会受当前环境变量设置影响。这可以避免在环境里同时存在多种凭据时，被意外路由到 Anthropic。

### 已测试的模型与别名

以下是内置别名表中已注册、并且有已知 token 限额的模型：

| 别名 | 解析后的模型名 | 提供商 | 最大输出 tokens | 上下文窗口 |
|---|---|---|---|---|
| `opus` | `claude-opus-4-6` | Anthropic | 32 000 | 200 000 |
| `sonnet` | `claude-sonnet-4-6` | Anthropic | 64 000 | 200 000 |
| `haiku` | `claude-haiku-4-5-20251213` | Anthropic | 64 000 | 200 000 |
| `grok` / `grok-3` | `grok-3` | xAI | 64 000 | 131 072 |
| `grok-mini` / `grok-3-mini` | `grok-3-mini` | xAI | 64 000 | 131 072 |
| `grok-2` | `grok-2` | xAI | — | — |

任何未命中别名的模型名都会原样透传。这也是你使用 OpenRouter 模型 slug（`openai/gpt-4.1-mini`）、Ollama 标签（`llama3.2`）或完整 Anthropic 模型 ID（`claude-sonnet-4-20250514`）的方式。

### 用户自定义别名

你可以在任意一个设置文件中添加自定义别名（`~/.claw/settings.json`、`.claw/settings.json` 或 `.claw/settings.local.json`）：

```json
{
  "aliases": {
    "fast": "claude-haiku-4-5-20251213",
    "smart": "claude-opus-4-6",
    "cheap": "grok-3-mini"
  }
}
```

本地项目设置会覆盖用户级设置。别名也会继续通过内置别名表解析，因此 `"fast": "haiku"` 同样可行。

### 提供商检测的工作方式

1. 如果解析后的模型名以 `claude` 开头 → 选择 Anthropic
2. 如果以 `grok` 开头 → 选择 xAI
3. 否则，`claw` 会检查当前设置了哪种凭据：先看 `ANTHROPIC_API_KEY` / `ANTHROPIC_AUTH_TOKEN`，然后是 `OPENAI_API_KEY`，最后是 `XAI_API_KEY`
4. 如果都不匹配，则默认使用 Anthropic

## 常见问题

### Codex 是什么？

“codex” 这个词会出现在 Claw Code 生态里，但它**不是**指 OpenAI Codex（那个代码生成模型）。在这个项目里，它的含义是：

- **`oh-my-codex`（OmX）** 是构建在 `claw` 之上的工作流与插件层。它提供规划模式、并行多代理执行、通知路由和其他自动化能力。参见 [PHILOSOPHY_zh.md](./PHILOSOPHY_zh.md) 和 [oh-my-codex 仓库](https://github.com/Yeachan-Heo/oh-my-codex)。
- **`.codex/` 目录**（例如 `.codex/skills`、`.codex/agents`、`.codex/commands`）是遗留查找路径，`claw` 仍会和主路径 `.claw/` 一起扫描它们。
- **`CODEX_HOME`** 是一个可选环境变量，用来指定用户级 skill 和 command 查找的自定义根目录。

`claw` **不支持** OpenAI Codex session、Codex CLI 或 Codex session 的导入/导出。如果你需要使用 OpenAI 模型（例如 GPT-4.1），请按照上文 [OpenAI 兼容端点](#openai-compatible-endpoint) 和 [OpenRouter](#openrouter) 章节中的方式来配置 OpenAI-compatible provider。

## HTTP 代理支持

当 `claw` 向 Anthropic、OpenAI-compatible 和 xAI-compatible 端点发起出站请求时，会自动遵循标准的 `HTTP_PROXY`、`HTTPS_PROXY` 和 `NO_PROXY` 环境变量（接受大写和小写形式）。在启动 CLI 之前设置它们，底层的 `reqwest` client 就会自动完成配置。

### 环境变量

```bash
export HTTPS_PROXY="http://proxy.corp.example:3128"
export HTTP_PROXY="http://proxy.corp.example:3128"
export NO_PROXY="localhost,127.0.0.1,.corp.example"

cd rust
./target/debug/claw prompt "hello via the corporate proxy"
```

### 编程式 `proxy_url` 配置项

除了按协议分别设置环境变量之外，`ProxyConfig` 类型还提供了一个 `proxy_url` 字段，可以作为 HTTP 和 HTTPS 的统一代理入口。当设置了 `proxy_url` 时，它会优先于单独的 `http_proxy` 和 `https_proxy` 字段。

```rust
use api::{build_http_client_with, ProxyConfig};

// 从一个统一 URL 构建（例如来自配置文件、CLI 参数等）
let config = ProxyConfig::from_proxy_url("http://proxy.corp.example:3128");
let client = build_http_client_with(&config).expect("proxy client");

// 或直接设置字段，并搭配 NO_PROXY
let config = ProxyConfig {
    proxy_url: Some("http://proxy.corp.example:3128".to_string()),
    no_proxy: Some("localhost,127.0.0.1".to_string()),
    ..ProxyConfig::default()
};
let client = build_http_client_with(&config).expect("proxy client");
```

### 说明

- 当同时设置了 `HTTPS_PROXY` 和 `HTTP_PROXY` 时，安全代理用于 `https://` URL，普通代理用于 `http://` URL。
- `proxy_url` 是一个统一替代项：设置后会同时作用于 `http://` 与 `https://` 目标，并覆盖分协议字段。
- `NO_PROXY` 接受逗号分隔的主机后缀列表（例如 `.corp.example`）以及 IP 字面量。
- 空值会被视为未设置，因此 `HTTPS_PROXY=""` 不会启用代理。
- 如果代理 URL 无法解析，`claw` 会回退到直连（无代理）客户端，以保证现有工作流仍可继续；如果你原本期望请求走隧道，请再次检查 URL 是否正确。

## 常用运维命令

```bash
cd rust
./target/debug/claw status
./target/debug/claw sandbox
./target/debug/claw agents
./target/debug/claw mcp
./target/debug/claw skills
./target/debug/claw system-prompt --cwd .. --date 2026-04-04
```

## Session 管理

REPL 的回合记录会保存在当前工作区下的 `.claw/sessions/` 中。

```bash
cd rust
./target/debug/claw --resume latest
./target/debug/claw --resume latest /status /diff
```

常用的交互命令包括 `/help`、`/status`、`/cost`、`/config`、`/session`、`/model`、`/permissions` 和 `/export`。

## 配置文件解析顺序

运行时配置按以下顺序加载，后面的条目会覆盖前面的条目：

1. `~/.claw.json`
2. `~/.config/claw/settings.json`
3. `<repo>/.claw.json`
4. `<repo>/.claw/settings.json`
5. `<repo>/.claw/settings.local.json`

## Mock parity harness

工作区中包含一个确定性的 Anthropic 兼容 mock 服务与 parity harness。

```bash
cd rust
./scripts/run_mock_parity_harness.sh
```

手动启动 mock 服务：

```bash
cd rust
cargo run -p mock-anthropic-service -- --bind 127.0.0.1:0
```

## 验证

```bash
cd rust
cargo test --workspace
```

## 工作区概览

当前 Rust crates：

- `api`
- `commands`
- `compat-harness`
- `mock-anthropic-service`
- `plugins`
- `runtime`
- `rusty-claude-cli`
- `telemetry`
- `tools`
