# 以容器为优先的 claw-code 工作流

在新增本文档之前，本仓库的 Rust 运行时就已经具备了**容器检测**能力：

- `rust/crates/runtime/src/sandbox.rs` 会检测 Docker/Podman/容器标记，例如 `/.dockerenv`、`/run/.containerenv`、匹配的环境变量，以及 `/proc/1/cgroup` 中的线索。
- `rust/crates/rusty-claude-cli/src/main.rs` 通过 `claw sandbox` / `cargo run -p rusty-claude-cli -- sandbox` 报告暴露这类状态。
- `.github/workflows/rust-ci.yml` 在 `ubuntu-latest` 上运行，但它**并没有**定义 Docker 或 Podman 容器作业。
- 在这次变更之前，仓库中**没有**提交 `Dockerfile`、`Containerfile` 或 `.devcontainer/` 配置。

本文档新增了一个很小的、已提交到仓库的 `Containerfile`，让 Docker 和 Podman 用户都能有一套规范的容器工作流。

## 这个已提交容器镜像的用途

仓库根目录下的 [`../Containerfile`](../Containerfile) 为你提供了一个可复用的 Rust 构建/测试 shell，并预装了这个工作区常用的额外包（`git`、`pkg-config`、`libssl-dev`、证书）。

它**不会**把仓库复制进镜像。相反，推荐做法是把你的检出目录通过 bind-mount 挂载到 `/workspace`，这样编辑仍然保留在宿主机上。

## 构建镜像

在仓库根目录执行：

### Docker

```bash
docker build -t claw-code-dev -f Containerfile .
```

### Podman

```bash
podman build -t claw-code-dev -f Containerfile .
```

## 在容器中运行 `cargo test --workspace`

下面这些命令会挂载仓库、避免把 Cargo 构建产物写进工作树，并从 `rust/` 这个 Rust 工作区目录启动。

### Docker

```bash
docker run --rm -it \
  -v "$PWD":/workspace \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev \
  cargo test --workspace
```

### Podman

```bash
podman run --rm -it \
  -v "$PWD":/workspace:Z \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev \
  cargo test --workspace
```

如果你想做一次完全干净的重建，可以在 `cargo test --workspace` 前加上 `cargo clean &&`。

## 在容器中打开一个 shell

### Docker

```bash
docker run --rm -it \
  -v "$PWD":/workspace \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev
```

### Podman

```bash
podman run --rm -it \
  -v "$PWD":/workspace:Z \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev
```

进入 shell 后：

```bash
cargo build --workspace
cargo test --workspace
cargo run -p rusty-claude-cli -- --help
cargo run -p rusty-claude-cli -- sandbox
```

`sandbox` 命令是一个很实用的健全性检查：在 Docker 或 Podman 内部，它应该报告 `In container true`，并列出运行时检测到的容器标记。

## 同时挂载本仓库和另一个仓库

如果你希望在保持 `claw-code` 本身读写挂载的同时，用 `claw` 操作另一个检出目录：

### Docker

```bash
docker run --rm -it \
  -v "$PWD":/workspace \
  -v "$HOME/src/other-repo":/repo \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev
```

### Podman

```bash
podman run --rm -it \
  -v "$PWD":/workspace:Z \
  -v "$HOME/src/other-repo":/repo:Z \
  -e CARGO_TARGET_DIR=/tmp/claw-target \
  -w /workspace/rust \
  claw-code-dev
```

例如：

```bash
cargo run -p rusty-claude-cli -- prompt "summarize /repo"
```

## 说明

- Docker 和 Podman 共用同一个已提交的 `Containerfile`
- Podman 示例中的 `:Z` 后缀用于 SELinux 重新标记；在 Fedora/RHEL 系主机上请保留
- 使用 `CARGO_TARGET_DIR=/tmp/claw-target` 可以避免在 bind-mount 的检出目录里留下属于容器用户的 `target/` 产物
- 如果你是在本地、非容器环境中开发，请继续参考 [`../USAGE_zh.md`](../USAGE_zh.md) 和 [`../rust/README.md`](../rust/README.md)
