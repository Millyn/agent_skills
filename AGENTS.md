# AGENTS.md — Agent 软件开发工作指南

## 核心原则

本项目遵循 `agent-dev-workflow` skill 规范。详细内容请查阅：

- **Skill 位置**：`.claude/skills/agent-dev-workflow/SKILL.md`
- **完整规范**：`.claude/skills/agent-dev-workflow/references/`

## Session 层级架构

本项目采用 **主 Session → 子 Session → 子 Agent** 的多层级架构：

```
主 Session（流程统筹 Agent）
    │
    ├── 拆解需求
    ├── 创建子 Session
    │
    └── 子 Session A
        │
        ├── 子 Session 主 Agent（规划下发）
        │
        ├── 子 Agent 1（执行任务）
        ├── 子 Agent 2（执行任务）
        └── 子 Agent N...
```

### 核心概念

- **主 Session**：负责整体需求拆解、任务规划、创建子 Session、最终验收和交付
- **子 Session**：负责落地主 Session 拆解下来的可执行任务，是任务执行的容器；在 GUI 多会话环境中必须创建为真实可视化的独立会话
- **子 Session 主 Agent**：每个子 Session 内部的规划者，负责将子 Session 任务进一步拆解并下发给子 Agent
- **子 Agent**：负责执行具体任务的执行者

### Session 与 Agent 的区别

| 维度 | Session | Agent |
|------|---------|-------|
| 定位 | 任务执行容器 | 任务执行者 |
| 角色 | 无多角色区分，只负责落地 | 有多种角色（统筹、执行、测试） |
| 职责 | 执行主 Session 拆解的可执行任务 | 执行具体的技术任务 |
| 数量 | 可有多个子 Session | 每个子 Session 内可有多个子 Agent |

## 快速参考

### 开发流程

```
需求理解 → 项目分析 → GitHub 调研 → 需求拆解 → 创建子 Session → 子 Session 内部规划 → 子 Agent 执行 → 定向测试 → 验收 → 合并 → 交付
```

### 用户反馈修复流程

```
用户反馈 → 问题分析 → 创建子 Session → 子 Session 内部规划 → 子 Agent 修复 → 验收 → 交付
```

**重要原则**：用户校验后的 BUG 修复和功能完善，也必须按照规范流程分配给子 Session 处理，主 Session 不直接修复。

### 角色分工

#### 主 Session 层级
- **流程统筹 Agent（主 Session Agent）**：负责整体方向、需求拆解、创建子 Session、任务分配、验收和交付

#### 子 Session 层级
- **子 Session 主 Agent**：负责子 Session 内部的任务规划、拆解和下发给子 Agent
- **任务执行 Agent（子 Agent）**：负责执行子 Session 主 Agent 分配的具体任务，不越界
- **质量验证 Agent（测试 Agent）**：唯一的正式定向测试层，验证修改范围内的功能并产出测试报告，供各级验收复用

GUI 展示时使用上述稳定职能名称，并与当前任务状态同时显示，例如：`流程统筹 Agent｜工作中`、`Session:Session名称｜子 Session 主 Agent｜工作中`。

### 关键规则

1. 不臆想需求，有依据才行动
2. UPDATE.MD 是版本计划的事实来源
3. 默认只开发下一版本
4. 子 Session 必须定义明确的任务边界
5. 子 Agent 完成后必须提供标准化报告
6. 用户反馈的 BUG 和未完成内容，必须分配给子 Session 处理
7. 测试唯一责任制：同一修改范围的完整测试只执行一次，任务执行 Agent 只做最小自测，验收层复用测试结论不重跑

### Session 管理

- Session 不只是内部容器：Codex、opencode 等 GUI 多会话环境中，子 Session 必须创建为真实、可视化、用户可见的独立会话
- 环境不支持多会话时才退化为内部子会话，并在 `.m-work-flow/sessions/` 完整留痕
- 专项 Session 原则：按业务域命名（`session-<领域>`），一个领域一个专项
- 复用优先：已验收的 Session 进入"待复用"并保留上下文；同领域新问题优先重新激活，不新建
- 主 Session 维护注册表：`.m-work-flow/sessions/registry.md`（名称、领域、状态、复用次数）

### 主 Session 任务边界格式

```
子 Session：<名称>
任务：<描述>
负责：<范围>
依赖：<其他子 Session>
修改范围：<实际允许变更的模块、文件或功能>
影响范围：<直接调用链、依赖模块、资源、数据或用户流程>
测试目标：<需要验证的行为和风险>
测试边界：<本次默认不覆盖的范围；如需全量回归需明确说明>
验收标准：<标准列表>
```

### 子 Session 任务边界格式

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

子 Session 主 Agent 分配任务时，必须根据实际任务填写"修改范围、影响范围、测试目标和测试边界"。质量验证 Agent 在用户未明确要求全量回归时，默认依据这些信息进行定向测试，不得仅因项目规模或惯例一味执行全量测试。测试范围扩大时必须记录扩大的范围及理由。

### 完成报告格式

#### 任务执行 Agent 完成报告格式
```
任务：<描述>
修改文件：<列表>
完成内容：<列表>
自测结果：<最小自测方式与结果；不要求覆盖影响链路>
发现的问题：<内容>
额外修改：<有/无>
需要子 Session 主 Agent 注意：<内容>
```

#### 质量验证 Agent 测试报告格式
```
测试对象：<修改范围和影响范围>
测试结果：<通过/失败及详情>
实际测试范围：<已执行的修改范围、直接影响链路和必要的最小冒烟验证>
未测试范围及原因：<未覆盖范围和原因；无则填写"无">
范围扩大及理由：<超出分配测试边界时必填；无则填写"无">
发现的问题：<内容>
```

#### 子 Session 完成报告格式
```
子 Session：<名称>
完成任务：<列表>
包含子 Agent：<列表>
测试情况：<质量验证 Agent 测试结论及来源；未单独分配质量验证 Agent 时记录简化决定和理由>
汇总结果：<内容>
需要主 Session 注意：<内容>
```

## 详细规范

如需了解详细规范，请查阅 skill 文档：

- 开发流程详情 → [references/development-workflow.md](.claude/skills/agent-dev-workflow/references/development-workflow.md)
- 版本管理规范 → [references/version-management.md](.claude/skills/agent-dev-workflow/references/version-management.md)
- Session 管理规范 → [references/session-management.md](.claude/skills/agent-dev-workflow/references/session-management.md)
- Agent 角色职责 → [references/agent-roles.md](.claude/skills/agent-dev-workflow/references/agent-roles.md)
- 测试规范 → [references/testing.md](.claude/skills/agent-dev-workflow/references/testing.md)
- 交付规范 → [references/delivery.md](.claude/skills/agent-dev-workflow/references/delivery.md)
