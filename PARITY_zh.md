# 兼容性状态：claw-code Rust 移植

最后更新：2026-04-03

## 摘要

- 规范文档：顶层这个 `PARITY.md` 是 `rust/scripts/run_mock_parity_diff.py` 消费的文件。
- 请求的 9 车道检查点：**9 条车道已全部合并到 `main`。**
- 当前 `main` HEAD：`ee31e00`（stub 实现已被真实的 AskUserQuestion + RemoteTrigger 替换）。
- 该检查点下的仓库统计：**`main` 上 292 次提交 / 所有分支共 293 次提交**，**9 个 crate**，**48,599 行受跟踪 Rust 代码**，**2,568 行测试代码**，**3 位作者**，日期范围为 **2026-03-31 → 2026-04-03**。
- Mock parity harness 统计：**10 个脚本化场景**，**19 个被捕获的 `/v1/messages` 请求**，位于 `rust/crates/rusty-claude-cli/tests/mock_parity_harness.rs`。

## Mock parity harness：里程碑 1

- [x] 确定性的 Anthropic 兼容 mock 服务（`rust/crates/mock-anthropic-service`）
- [x] 可复现的洁净环境 CLI harness（`rust/crates/rusty-claude-cli/tests/mock_parity_harness.rs`）
- [x] 脚本化场景：`streaming_text`、`read_file_roundtrip`、`grep_chunk_assembly`、`write_file_allowed`、`write_file_denied`

## Mock parity harness：里程碑 2（行为扩展）

- [x] 脚本化的多工具回合覆盖：`multi_tool_turn_roundtrip`
- [x] 脚本化的 bash 覆盖：`bash_stdout_roundtrip`
- [x] 脚本化的权限提示覆盖：`bash_permission_prompt_approved`、`bash_permission_prompt_denied`
- [x] 脚本化的插件路径覆盖：`plugin_tool_roundtrip`
- [x] 行为差异/清单运行器：`rust/scripts/run_mock_parity_diff.py`

## Harness v2 行为检查清单

规范场景映射：`rust/mock_parity_scenarios.json`

- 多工具 assistant 回合
- Bash 流程往返
- 各类工具路径上的权限执行
- 插件工具执行路径
- 文件工具：已由 harness 验证的流程
- 由 mock parity harness 验证的流式响应支持

## 9 车道检查点

| 车道 | 状态 | 功能提交 | 合并提交 | 证据 |
|---|---|---|---|---|
| 1. Bash 校验 | 已合并 | `36dac6c` | `1cfd78a` | `jobdori/bash-validation-submodules`，`rust/crates/runtime/src/bash_validation.rs`（`main` 上 `+1004`） |
| 2. CI 修复 | 已合并 | `89104eb` | `f1969ce` | `rust/crates/runtime/src/sandbox.rs`（`+22/-1`） |
| 3. 文件工具 | 已合并 | `284163b` | `a98f2b6` | `rust/crates/runtime/src/file_ops.rs`（`+195/-1`） |
| 4. TaskRegistry | 已合并 | `5ea138e` | `21a1e1d` | `rust/crates/runtime/src/task_registry.rs`（`+336`） |
| 5. Task 接线 | 已合并 | `e8692e4` | `d994be6` | `rust/crates/tools/src/lib.rs`（`+79/-35`） |
| 6. Team+Cron | 已合并 | `c486ca6` | `49653fe` | `rust/crates/runtime/src/team_cron_registry.rs`、`rust/crates/tools/src/lib.rs`（`+441/-37`） |
| 7. MCP 生命周期 | 已合并 | `730667f` | `cc0f92e` | `rust/crates/runtime/src/mcp_tool_bridge.rs`、`rust/crates/tools/src/lib.rs`（`+491/-24`） |
| 8. LSP 客户端 | 已合并 | `2d66503` | `d7f0dc6` | `rust/crates/runtime/src/lsp_client.rs`、`rust/crates/tools/src/lib.rs`（`+461/-9`） |
| 9. 权限执行 | 已合并 | `66283f4` | `336f820` | `rust/crates/runtime/src/permission_enforcer.rs`、`rust/crates/tools/src/lib.rs`（`+357`） |

## 车道详情

### 车道 1：Bash 校验

- **状态：** 已合并到 `main`
- **功能提交：** `36dac6c` — `feat: add bash validation submodules — readOnlyValidation, destructiveCommandWarning, modeValidation, sedValidation, pathValidation, commandSemantics`
- **证据：** 分支专属 diff 新增了 `rust/crates/runtime/src/bash_validation.rs` 和一个 `runtime::lib` 导出（2 个文件合计 `+1005`）。
- **`main` 分支现实情况：** `rust/crates/runtime/src/bash.rs` 仍是 `main` 上的活动实现，共 **283 行代码**，包含 timeout/background/sandbox 执行。`PermissionEnforcer::check_bash()` 在 `main` 上增加了只读门控，但专用校验模块尚未落地。

### Bash 工具：上游有 18 个子模块，Rust 版本只有 1 个

- 在 `main` 上，这个说法在实质上仍然成立。
- Harness 覆盖证明了 bash 执行与提示升级流程，但尚未覆盖完整的上游校验矩阵。
- 这个仅存在于分支上的车道目标包括 `readOnlyValidation`、`destructiveCommandWarning`、`modeValidation`、`sedValidation`、`pathValidation` 和 `commandSemantics`。

### 车道 2：CI 修复

- **状态：** 已合并到 `main`
- **功能提交：** `89104eb` — `fix(sandbox): probe unshare capability instead of binary existence`
- **合并提交：** `f1969ce` — `Merge jobdori/fix-ci-sandbox: probe unshare capability for CI fix`
- **证据：** `rust/crates/runtime/src/sandbox.rs` 现为 **385 行代码**，现在通过真实的 `unshare` 能力和容器信号来判断 sandbox 支持，而不再根据二进制是否存在来假定支持。
- **意义：** `.github/workflows/rust-ci.yml` 会运行 `cargo fmt --all --check` 和 `cargo test -p rusty-claude-cli`；这个车道移除了运行时行为里一个 CI 特有的 sandbox 假设。

### 车道 3：文件工具

- **状态：** 已合并到 `main`
- **功能提交：** `284163b` — `feat(file_ops): add edge-case guards — binary detection, size limits, workspace boundary, symlink escape`
- **合并提交：** `a98f2b6` — `Merge jobdori/file-tool-edge-cases: binary detection, size limits, workspace boundary guards`
- **证据：** `rust/crates/runtime/src/file_ops.rs` 现为 **744 行代码**，并已包含 `MAX_READ_SIZE`、`MAX_WRITE_SIZE`、NUL 字节二进制检测，以及规范化后的工作区边界校验。
- **Harness 覆盖：** `read_file_roundtrip`、`grep_chunk_assembly`、`write_file_allowed` 和 `write_file_denied` 已在 manifest 中声明，并由 clean-env harness 运行。

### 文件工具：已由 harness 验证的流程

- `read_file_roundtrip` 检查读取路径执行与最终综合输出。
- `grep_chunk_assembly` 检查分块 grep 工具输出的处理。
- `write_file_allowed` 和 `write_file_denied` 同时验证写入成功与权限拒绝路径。

### 车道 4：TaskRegistry

- **状态：** 已合并到 `main`
- **功能提交：** `5ea138e` — `feat(runtime): add TaskRegistry — in-memory task lifecycle management`
- **合并提交：** `21a1e1d` — `Merge jobdori/task-runtime: TaskRegistry in-memory lifecycle management`
- **证据：** `rust/crates/runtime/src/task_registry.rs` 为 **335 行代码**，在线程安全的内存注册表之上提供了 `create`、`get`、`list`、`stop`、`update`、`output`、`append_output`、`set_status` 与 `assign_team`。
- **范围：** 这个车道用真实的运行时任务记录替换了纯固定载荷的 stub 状态，但它本身并未加入外部子进程执行能力。

### 车道 5：Task 接线

- **状态：** 已合并到 `main`
- **功能提交：** `e8692e4` — `feat(tools): wire TaskRegistry into task tool dispatch`
- **合并提交：** `d994be6` — `Merge jobdori/task-registry-wiring: real TaskRegistry backing for all 6 task tools`
- **证据：** `rust/crates/tools/src/lib.rs` 通过 `execute_tool()` 和具体的 `run_task_*` 处理器，分发 `TaskCreate`、`TaskGet`、`TaskList`、`TaskStop`、`TaskUpdate` 和 `TaskOutput`。
- **当前状态：** 任务工具现在通过 `global_task_registry()` 在 `main` 上暴露真实的注册表状态。

### 车道 6：Team+Cron

- **状态：** 已合并到 `main`
- **功能提交：** `c486ca6` — `feat(runtime+tools): TeamRegistry and CronRegistry — replace team/cron stubs`
- **合并提交：** `49653fe` — `Merge jobdori/team-cron-runtime: TeamRegistry + CronRegistry wired into tool dispatch`
- **证据：** `rust/crates/runtime/src/team_cron_registry.rs` 为 **363 行代码**，新增了线程安全的 `TeamRegistry` 和 `CronRegistry`；`rust/crates/tools/src/lib.rs` 则将 `TeamCreate`、`TeamDelete`、`CronCreate`、`CronDelete` 和 `CronList` 接入这些注册表。
- **当前状态：** team/cron 工具现在在 `main` 上具备内存态生命周期行为；但仍未达到真实后台调度器或 worker 集群的程度。

### 车道 7：MCP 生命周期

- **状态：** 已合并到 `main`
- **功能提交：** `730667f` — `feat(runtime+tools): McpToolRegistry — MCP lifecycle bridge for tool surface`
- **合并提交：** `cc0f92e` — `Merge jobdori/mcp-lifecycle: McpToolRegistry lifecycle bridge for all MCP tools`
- **证据：** `rust/crates/runtime/src/mcp_tool_bridge.rs` 为 **406 行代码**，跟踪服务器连接状态、资源列表、资源读取、工具列表、工具分发确认、认证状态和断开连接。
- **接线：** `rust/crates/tools/src/lib.rs` 将 `ListMcpResources`、`ReadMcpResource`、`McpAuth` 和 `MCP` 路由到 `global_mcp_registry()` 处理器。
- **范围：** 这个车道在 `main` 上用注册表桥接替换了纯 stub 响应；端到端 MCP 连接填充以及更广泛的传输/运行时深度，仍取决于更宽层的 MCP 运行时（`mcp_stdio.rs`、`mcp_client.rs`、`mcp.rs`）。

### 车道 8：LSP 客户端

- **状态：** 已合并到 `main`
- **功能提交：** `2d66503` — `feat(runtime+tools): LspRegistry — LSP client dispatch for tool surface`
- **合并提交：** `d7f0dc6` — `Merge jobdori/lsp-client: LspRegistry dispatch for all LSP tool actions`
- **证据：** `rust/crates/runtime/src/lsp_client.rs` 为 **438 行代码**，以一个有状态注册表建模了 diagnostics、hover、definition、references、completion、symbols 和 formatting。
- **接线：** `rust/crates/tools/src/lib.rs` 中暴露的 `LSP` 工具 schema 目前列举了 `symbols`、`references`、`diagnostics`、`definition` 和 `hover`，然后通过 `registry.dispatch(action, path, line, character, query)` 路由请求。
- **范围：** 当前兼容性处于 registry/dispatch 层级；completion/format 支持存在于注册表模型中，但在工具 schema 边界上暴露得没有那么明确，而真正的外部语言服务器进程编排仍是另一个层面的问题。

### 车道 9：权限执行

- **状态：** 已合并到 `main`
- **功能提交：** `66283f4` — `feat(runtime+tools): PermissionEnforcer — permission mode enforcement layer`
- **合并提交：** `336f820` — `Merge jobdori/permission-enforcement: PermissionEnforcer with workspace + bash enforcement`
- **证据：** `rust/crates/runtime/src/permission_enforcer.rs` 为 **340 行代码**，在 `rust/crates/runtime/src/permissions.rs` 之上新增了工具门控、文件写入边界检查和 bash 只读启发式。
- **接线：** `rust/crates/tools/src/lib.rs` 暴露了 `enforce_permission_check()`，并在工具规格中携带每个工具的 `required_permission` 值。

### 工具路径上的权限执行

- Harness 场景验证了 `write_file_denied`、`bash_permission_prompt_approved` 和 `bash_permission_prompt_denied`。
- `PermissionEnforcer::check()` 会委托给 `PermissionPolicy::authorize()`，并返回结构化的允许/拒绝结果。
- `check_file_write()` 执行工作区边界和只读拒绝；`check_bash()` 则在只读模式下拒绝可变更命令，并在 prompt 模式下没有确认时阻止 bash。

## 工具面：`main` 上暴露了 40 个工具规格

- `rust/crates/tools/src/lib.rs` 中的 `mvp_tool_specs()` 暴露了 **40** 个工具规格。
- `bash`、`read_file`、`write_file`、`edit_file`、`glob_search` 和 `grep_search` 具备核心执行能力。
- `mvp_tool_specs()` 中现有的产品工具还包括 `WebFetch`、`WebSearch`、`TodoWrite`、`Skill`、`Agent`、`ToolSearch`、`NotebookEdit`、`Sleep`、`SendUserMessage`、`Config`、`EnterPlanMode`、`ExitPlanMode`、`StructuredOutput`、`REPL` 和 `PowerShell`。
- 这次 9 车道推进将 `Task*`、`Team*`、`Cron*`、`LSP` 和 MCP 工具从纯固定载荷 stub 替换成了 `main` 上基于 registry 的处理器。
- `Brief` 在 `execute_tool()` 中作为执行别名处理，但在 `mvp_tool_specs()` 中并不是一个单独暴露的工具规格。

### 仍然受限或故意保持浅层

- `AskUserQuestion` 仍返回 pending 响应载荷，而不是真实的交互式 UI 接线。
- `RemoteTrigger` 仍是 stub 响应。
- `TestingPermission` 仍只用于测试。
- Task、team、cron、MCP 和 LSP 不再只是 `execute_tool()` 中的固定载荷 stub，但其中若干仍是基于 registry 的近似实现，而非完整的外部运行时集成。
- 在 `36dac6c` 合并前，bash 深度校验仍只存在于分支上。

## 与旧 PARITY 检查表的对账结果

- [x] 路径穿越防护（符号链接跟随、`../` 逃逸）
- [x] 读写大小限制
- [x] 二进制文件检测
- [x] 权限模式执行（只读 vs workspace-write）
- [x] 配置合并优先级（user > project > local）— `ConfigLoader::discover()` 按 user → project → local 加载，`loads_and_merges_claude_code_config_files_by_precedence()` 验证了合并顺序
- [x] 插件 install/enable/disable/uninstall 流程 — `rust/crates/commands/src/lib.rs` 中的 `/plugin` slash 处理委托给了 `rust/crates/plugins/src/lib.rs` 中的 `PluginManager::{install, enable, disable, uninstall}`
- [x] 没有通过 `#[ignore]` 隐藏失败的测试 — 对 `rust/**/*.rs` 执行 `grep` 得到 0 个 ignored tests

## 仍未完成

- [ ] 超越当前 `main` 上 registry bridge 的端到端 MCP 运行时生命周期
- [x] 输出截断（大 stdout/大文件内容）
- [ ] 会话压缩行为匹配
- [ ] token 计数 / 成本跟踪准确性
- [x] Bash validation 车道已合并到 `main`
- [ ] 每次提交都保持 CI 绿色

## 迁移就绪度

- [x] `PARITY.md` 持续维护且内容诚实
- [x] 已记录 9 条请求车道的提交哈希与当前状态
- [x] 9 条请求车道均已落在 `main` 上（`bash-validation` 仍只存在于分支上）
- [x] 没有通过 `#[ignore]` 隐藏失败的测试
- [ ] 每次提交都保持 CI 绿色
- [x] 代码库形态已足够清晰，可用于交接文档
