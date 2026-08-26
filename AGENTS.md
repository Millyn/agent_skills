# AGENTS.md — Agent 软件开发工作指南

## 核心原则

本项目遵循 `agent-dev-workflow` skill 规范：

- **Skill 位置**：`.claude/skills/agent-dev-workflow/SKILL.md`
- **完整规范**：`.claude/skills/agent-dev-workflow/references/`

本文件是常驻速查卡；细节以 references 为准，各主题只有一个权威来源。

## Thread 层级架构

本项目采用 **主 Thread → 子 Thread → 子 Agent** 的多层级架构：

```
主 Thread（流程统筹 Agent）
    │
    ├── 拆解需求
    ├── 通过 create_thread 创建子 Thread
    │
    └── 子 Thread A
        │
        ├── 子 Thread 主 Agent（规划下发）
        │
        ├── 子 Agent 1（执行任务）
        ├── 子 Agent 2（执行任务）
        └── 子 Agent N...
```

### 核心概念

- **主 Thread**：整体需求拆解、任务规划、创建子 Thread、最终验收和交付
- **子 Thread**：任务执行的容器；主 Thread 必须使用 `create_thread` 命令将其创建为真实可视化的独立 Thread
- **子 Thread 主 Agent**：子 Thread 内部的规划者，将任务进一步拆解并下发给子 Agent
- **子 Agent**：具体任务的执行者

### Thread 与 Agent 的区别

| 维度 | Thread | Agent |
|------|---------|-------|
| 定位 | 任务执行容器 | 任务执行者 |
| 角色 | 无多角色区分，只负责落地 | 有多种角色（统筹、执行、测试） |
| 数量 | 可有多个子 Thread | 每个子 Thread 内可有多个子 Agent |
| 创建方式 | 主 Thread 使用 `create_thread` 命令创建 | 在所属 Thread 内部由上级 Agent 规划下发，不使用 create_thread |

**关键区分**：开启新的执行容器 = 主 Thread 调用 `create_thread` 创建新的子 Thread；在同一容器内增加执行者 = 分配子 Agent。Agent 不通过 `create_thread` 创建，也无需为每个子 Agent 单独创建 Thread。

## 快速参考

### 开发流程

```
需求理解 → 项目分析 → GitHub 调研 → 需求拆解 → 创建子 Thread → 子 Thread 内部规划 → 子 Agent 执行 → 定向测试 → 验收 → 合并 → 交付
```

### 用户反馈修复流程

```
用户反馈 → 问题分析 → 创建子 Thread → 子 Thread 内部规划 → 子 Agent 修复 → 验收 → 交付
```

**重要原则**：用户校验后的 BUG 修复和功能完善，也必须按照规范流程分配给子 Thread 处理，主 Thread 不直接修复。

### 关键规则

1. 不臆想需求，有依据才行动；不确定时停止并汇报
2. UPDATE.MD 是版本计划的事实来源，默认只开发下一版本
3. 子 Thread 必须定义明确的任务边界（含修改范围、影响范围、测试目标、测试边界）
4. 子 Agent 完成后必须提供标准化报告
5. 用户反馈的 BUG 和未完成内容，必须分配给子 Thread 处理
6. 测试唯一责任制：同一修改范围的完整测试只执行一次；任务执行 Agent 只做最小自测，质量验证 Agent 是唯一正式定向测试层，各级验收复用结论不重跑
7. 开始多 Thread 协作时创建 `.m-work-flow` 目录并加入 `.gitignore`

### Thread 管理要点

- 支持 `create_thread` 的 GUI 环境（Codex、opencode 等）：主 Thread 必须通过 `create_thread` 将子 Thread 创建为真实、可视化、用户可见的独立 Thread；不支持时才退化为内部子 Thread 并在 `.m-work-flow/threads/` 留痕
- 专项 Thread 按业务域命名（`thread-<领域>`），一个领域一个；已验收的进入"待复用"，同领域新问题优先重新激活，不新建
- 主 Thread 维护注册表 `.m-work-flow/threads/registry.md`（名称、领域、状态、复用次数）

### 角色分工

- **流程统筹 Agent（主 Thread）**：整体方向、需求拆解、创建子 Thread、任务分配、验收和交付
- **子 Thread 主 Agent**：子 Thread 内部的任务规划、拆解和下发给子 Agent
- **任务执行 Agent（子 Agent）**：执行分配的具体任务，不越界
- **质量验证 Agent**：唯一的正式定向测试层，验证修改范围内的功能并产出测试报告，供各级验收复用

GUI 展示固定使用上述稳定职能名称，并与当前任务状态同时显示，例如：`流程统筹 Agent｜工作中`、`Thread:<Thread名称>｜子 Thread 主 Agent｜工作中`。

### 主 Thread 任务边界格式

```
子 Thread：<名称>
任务：<描述>
负责：<范围>
依赖：<其他子 Thread>
修改范围：<实际允许变更的模块、文件或功能>
影响范围：<直接调用链、依赖模块、资源、数据或用户流程>
测试目标：<需要验证的行为和风险>
测试边界：<本次默认不覆盖的范围；如需全量回归需明确说明>
验收标准：<标准列表>
```

### 子 Thread 任务边界格式

```
任务：<描述>
负责：<范围>
允许修改：<文件列表>
禁止修改：<文件列表>
依赖：<依赖项>
修改范围：<实际允许变更的模块、文件或功能>
影响范围：<直接调用链、依赖模块、资源、数据或用户流程>
测试目标：<需要验证的行为和风险>
测试边界：<本次默认不覆盖的范围；如需全量回归需明确说明>
验收标准：<标准列表>
```

子 Thread 主 Agent 分配任务时必须填写四项测试相关信息；质量验证 Agent 默认据此定向测试，不因项目规模或惯例执行全量回归，扩大测试范围时必须记录理由。

### 完成报告格式

任务执行 Agent 完成报告、质量验证 Agent 测试报告、子 Thread 完成报告的权威模板统一见 [agent-roles.md](.claude/skills/agent-dev-workflow/references/agent-roles.md)。所有报告必须真实反映实际修改。

## 详细规范

如需了解详细规范，请查阅 skill 文档：

- 开发流程详情（含超时问询机制、.m-work-flow 目录） → [references/development-workflow.md](.claude/skills/agent-dev-workflow/references/development-workflow.md)
- 版本管理规范 → [references/version-management.md](.claude/skills/agent-dev-workflow/references/version-management.md)
- Thread 管理规范 → [references/thread-management.md](.claude/skills/agent-dev-workflow/references/thread-management.md)
- Agent 角色职责（含任务边界与报告模板权威版） → [references/agent-roles.md](.claude/skills/agent-dev-workflow/references/agent-roles.md)
- 测试规范 → [references/testing.md](.claude/skills/agent-dev-workflow/references/testing.md)
- 交付规范 → [references/delivery.md](.claude/skills/agent-dev-workflow/references/delivery.md)
