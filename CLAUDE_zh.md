# CLAUDE.md

本文档为 Claude Code（claude.ai/code）在处理本仓库代码时提供说明。

## 检测到的技术栈

- 语言：Rust
- 框架：未从支持的 starter 标记中检测到框架

## 验证

- 在 `rust/` 目录下运行 Rust 验证：`cargo fmt`、`cargo clippy --workspace --all-targets -- -D warnings`、`cargo test --workspace`
- `src/` 和 `tests/` 同时存在；当行为发生变化时，请同步更新这两个面向

## 仓库结构

- `rust/` 包含 Rust 工作区以及当前活跃的 CLI/运行时实现
- `src/` 包含需要与生成式说明和测试保持一致的源文件
- `tests/` 包含验证层，应与代码改动一起审查

## 协作约定

- 优先做小而易审查的改动，并让生成的引导文件与仓库的真实工作流保持一致
- 将共享默认值放在 `.claude.json` 中；将 `.claude/settings.local.json` 留给机器本地覆盖项
- 不要自动覆盖已有的 `CLAUDE.md` 内容；只有在仓库工作流发生变化时才有意识地更新它
