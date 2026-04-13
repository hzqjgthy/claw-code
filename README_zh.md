# Claw Code

<p align="center">
  <a href="https://github.com/ultraworkers/claw-code">ultraworkers/claw-code</a>
  ·
  <a href="./USAGE.md">使用说明</a>
  ·
  <a href="./rust/README.md">Rust 工作区</a>
  ·
  <a href="./PARITY.md">兼容性对齐</a>
  ·
  <a href="./ROADMAP.md">路线图</a>
  ·
  <a href="https://discord.gg/5TUQKqFWd">UltraWorkers Discord</a>
</p>

<p align="center">
  <a href="https://star-history.com/#ultraworkers/claw-code&Date">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=ultraworkers/claw-code&type=Date&theme=dark" />
      <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=ultraworkers/claw-code&type=Date" />
      <img alt="ultraworkers/claw-code 的 Star 历史" src="https://api.star-history.com/svg?repos=ultraworkers/claw-code&type=Date" width="600" />
    </picture>
  </a>
</p>

<p align="center">
  <img src="assets/claw-hero.jpeg" alt="Claw Code" width="300" />
</p>

Claw Code 是 `claw` CLI 代理运行框架的公开 Rust 实现。
规范实现位于 [`rust/`](./rust)，本仓库当前的权威来源是 **ultraworkers/claw-code**。

> [!IMPORTANT]
> 请先阅读 [`USAGE.md`](./USAGE.md)，了解构建、认证、CLI、会话和 parity-harness 工作流。构建完成后，先运行一次 `claw doctor` 作为健康检查；crate 级别的细节请参考 [`rust/README.md`](./rust/README.md)；当前 Rust 移植进度请查看 [`PARITY.md`](./PARITY.md)；如果你想使用容器优先的工作流，请阅读 [`docs/container.md`](./docs/container.md)。

## 当前仓库结构

- **`rust/`**：规范 Rust 工作区以及 `claw` CLI 二进制
- **`USAGE.md`**：面向任务的当前产品使用指南
- **`PARITY.md`**：Rust 移植的兼容性对齐状态与迁移说明
- **`ROADMAP.md`**：当前路线图与待清理事项
- **`PHILOSOPHY.md`**：项目目标与系统设计思路
- **`src/` + `tests/`**：配套的 Python/参考工作区与审计辅助工具；不是主要运行时入口

## 快速开始

> [!NOTE]
> [!WARNING]
> **`cargo install claw-code` 会安装错误的内容。** crates.io 上的 `claw-code` crate 是一个已弃用的占位包，它安装的是 `claw-code-deprecated.exe`，而不是 `claw`。运行后只会输出 `"claw-code has been renamed to agent-code"`。**不要使用 `cargo install claw-code`。** 你应该从源码构建（本仓库），或者安装上游二进制：
> ```bash
> cargo install agent-code   # 上游二进制：安装的是 'agent.exe'（Windows）/ 'agent'（Unix），不是 'agent-code'
> ```
> 本仓库（`ultraworkers/claw-code`）**仅支持从源码构建**，请按下列步骤操作。

```bash
# 1. 克隆并构建
git clone https://github.com/ultraworkers/claw-code
cd claw-code/rust
cargo build --workspace

# 2. 设置 API Key（Anthropic API Key，不是 Claude 订阅账号）
export ANTHROPIC_API_KEY="sk-ant-..."

# 3. 验证整体配置是否正确
./target/debug/claw doctor

# 4. 运行一个提示词
./target/debug/claw prompt "say hello"
```

> [!NOTE]
> **Windows（PowerShell）：** 二进制名称是 `claw.exe`，不是 `claw`。请使用 `.\target\debug\claw.exe`，或者运行 `cargo run -- prompt "say hello"` 以跳过路径查找。

### Windows 设置

**PowerShell 是受支持的 Windows 使用方式。** 你可以使用自己习惯的 shell。Windows 上最常见的入门问题如下：

1. **先安装 Rust**：从 <https://rustup.rs/> 下载并运行安装器。安装完成后，关闭并重新打开终端。
2. **确认 Rust 已加入 PATH：**
   ```powershell
   cargo --version
   ```
   如果失败，请重新打开终端，或者根据 Rust 安装器输出中的提示手动设置 PATH，然后再试一次。
3. **克隆并构建**（可在 PowerShell、Git Bash 或 WSL 中执行）：
   ```powershell
   git clone https://github.com/ultraworkers/claw-code
   cd claw-code/rust
   cargo build --workspace
   ```
4. **运行**（PowerShell，注意 `.exe` 和反斜杠）：
   ```powershell
   $env:ANTHROPIC_API_KEY = "sk-ant-..."
   .\target\debug\claw.exe prompt "say hello"
   ```

**Git Bash / WSL** 只是可选方案，不是必须。如果你更喜欢 bash 风格路径（例如 `/c/Users/you/...` 而不是 `C:\Users\you\...`），Git Bash（随 Git for Windows 一起提供）会很好用。在 Git Bash 中看到 `MINGW64` 提示符是正常现象，并不代表安装有问题。

> [!NOTE]
> **认证：** claw 需要使用 **API Key**（如 `ANTHROPIC_API_KEY`、`OPENAI_API_KEY` 等），不支持通过 Claude 订阅登录进行认证。

运行工作区测试套件：

```bash
cd rust
cargo test --workspace
```

## 文档导航

- [`USAGE.md`](./USAGE.md)：常用命令、认证、会话、配置、兼容性测试框架
- [`rust/README.md`](./rust/README.md)：crate 结构、CLI 能力、功能特性、工作区布局
- [`PARITY.md`](./PARITY.md)：Rust 移植的兼容性对齐状态
- [`rust/MOCK_PARITY_HARNESS.md`](./rust/MOCK_PARITY_HARNESS.md)：确定性 mock 服务测试框架说明
- [`ROADMAP.md`](./ROADMAP.md)：当前路线图与待处理清理工作
- [`PHILOSOPHY.md`](./PHILOSOPHY.md)：项目存在的原因与运作方式

## 生态

Claw Code 与更广泛的 UltraWorkers 工具链一起，以开放协作的方式持续构建：

- [clawhip](https://github.com/Yeachan-Heo/clawhip)
- [oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)
- [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)
- [oh-my-codex](https://github.com/Yeachan-Heo/oh-my-codex)
- [UltraWorkers Discord](https://discord.gg/5TUQKqFWd)

## 所有权 / 关联声明

- 本仓库**不声称拥有**原始 Claude Code 源材料的所有权。
- 本仓库**与 Anthropic 无关联，亦未获得其认可，也不是由其维护**。
