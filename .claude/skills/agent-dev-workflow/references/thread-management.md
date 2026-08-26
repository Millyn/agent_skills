# Thread 定义与管理规范

## Thread 的定义

Thread 是**任务执行的容器**，同时是用户可感知、可操作的工作单元。Thread 不只是内部逻辑概念：

- 在支持 `create_thread` 的 GUI 多 Thread 环境（Codex、opencode 等）中，主 Thread 必须使用 `create_thread` 命令将子 Thread 创建为**真实、可视化、用户可见的独立 Thread**
- 用户可以直接查看、切换、追问任何子 Thread
- 在不具备多 Thread 能力的环境中，才允许退化为内部子 Thread，但必须留痕

## Thread 形态

| 形态 | 适用环境 | 要求 |
|------|---------|------|
| 可视化 Thread | Codex、opencode 等支持 `create_thread` 的 GUI 多 Thread 环境 | **必须**通过 `create_thread` 命令在界面中创建真实 Thread；使用稳定命名；状态对用户可见 |
| 内部子 Thread | CLI 或不支持 `create_thread` 的环境 | 在 `.m-work-flow/threads/<thread名称>/` 中建立对应目录，任务边界、报告、通信记录全部落盘 |

### 判定规则

1. 先判断当前环境是否提供 `create_thread` 等创建真实 Thread 的能力。
2. 支持 → 必须可视化，不得在内部偷偷模拟。
3. 不支持 → 内部化 + 文件留痕，保证事后可追溯、可交接。
4. 无法判定时，向用户确认。

### 子 Agent 层级不受影响

无论 Thread 采用哪种形态，子 Thread 内部"子 Thread 主 Agent → 任务执行 Agent / 质量验证 Agent"的调度关系不变；子 Agent 由所在 Thread 内部调度，不强制每个子 Agent 都是独立 Thread。

## create_thread 使用规则

- 主 Thread 开启子 Thread 相当于调用 `create_thread` 命令创建一个真实、独立的 Thread；凡是需要新的执行容器，就必须使用 `create_thread`
- Agent 不通过 `create_thread` 创建：子 Thread 主 Agent、任务执行 Agent、质量验证 Agent 都位于所属 Thread 内部，由上级 Agent 通过标准任务边界格式下发产生
- 一个子 Thread 内可以包含多个子 Agent；不需要为每个子 Agent 单独调用 `create_thread`
- 仅当环境不支持 `create_thread` 或同类创建能力时，才退化为内部子 Thread，并在 `.m-work-flow/threads/` 中完整留痕

## 专项 Thread 与复用

### 专项原则

- 按业务域或问题域建立**专项 Thread**，例如：`thread-翻译功能`、`thread-数据同步`
- 一个领域一个专项 Thread，避免碎片化新建
- 同一领域的后续 BUG 修复、功能完善，优先复用已有专项 Thread

### 生命周期

```
创建 → 执行中 → 已验收 → 待复用 ──→ 重新激活 → 执行中 → ...
                      │
                      └──→ 关闭（用户明确要求或领域废弃）
```

- **已验收 ≠ 销毁**：完成验收的 Thread 默认进入"待复用"，保留全部上下文
- **重新激活**：主 Thread 向原 Thread 下发新的标准任务边界即可，沿用既有上下文
- **关闭条件**：仅当用户明确要求关闭，或该业务域永久废弃时才关闭

### 复用的收益

- 上下文延续：不重复分析项目结构、不重读代码
- 测试结论延续：上一轮质量验证报告可直接引用（衔接测试唯一责任制）
- 历史可追溯：用户可在原 Thread 直接查看历史决策和修改记录
- 降低复杂度：减少新建 Thread 带来的拆解、介绍和上下文重建成本

### 复用规则

1. 新建 Thread 前，必须先查 `.m-work-flow/threads/registry.md`，确认是否已有同领域待复用 Thread。
2. 有 → 优先重新激活复用；无 → 新建。
3. 复用时必须在原 Thread 中下发标准任务边界格式的新任务，不得沿用过期边界。
4. 跨领域的紧急小修复，经用户同意后可新建一次性 Thread，完成后按领域归属决定是否纳入注册表。

## Thread 注册表

主 Thread 必须维护 `.m-work-flow/threads/registry.md`，并在创建、激活、验收、关闭时更新：

```markdown
# Thread 注册表

| Thread 名称 | 领域 | 状态 | 创建时间 | 最近活动 | 复用次数 | 备注 |
|-------------|------|------|----------|----------|----------|------|
| thread-翻译功能 | 翻译 | 待复用 | <日期> | <日期> | 2 | 悬停+自动翻译 |
```

状态取值：`执行中`、`已验收`、`待复用`、`已关闭`

## 命名规范

- 格式：`thread-<领域名>`，例如 `thread-翻译功能`
- 名称必须稳定、语义清晰，复用时不得变更
- 同一领域同时只允许一个活跃专项 Thread

## GUI 展示要求

- 可视化 Thread 的 Thread 名称必须使用注册表中的稳定名称，例如：`Thread:thread-翻译功能｜任务执行 Agent｜工作中`
- 主 Thread 向用户展示各子 Thread 进度时，使用注册表中的状态值；角色展示名称规范见 [agent-roles.md](agent-roles.md)
