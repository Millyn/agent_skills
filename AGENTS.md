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
- **子 Thread**：任务执行的容器；主 Thread 必须使用 `create_thread` 命令将其创建为真实可视化的独立 Thread；`create_thread` 失败时按降级链处理（见 Thread 管理要点）
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
需求理解 → 项目分析 → GitHub 调研 → 需求拆解 → 创建子 Thread → 子 Thread 内部规划 → 子 Agent 执行 → 子 Agent 自测 → 汇报主 Thread → 主 Thread 安排验证 → 验收 → 合并 → 交付
```

### 用户反馈修复流程

```
用户反馈 → 问题分析 → 创建子 Thread → 子 Thread 内部规划 → 子 Agent 修复 → 子 Agent 自测 → 汇报主 Thread → 主 Thread 安排验证 → 验收 → 交付
```

**验收不通过时**：打回/退回 → 修复 → 自测 → 重新验收；累计 3 轮不通过 → 创建新 Thread 独立修复。

**重要原则**：用户校验后的 BUG 修复和功能完善，也必须按照规范流程分配给子 Thread 处理，主 Thread 不直接修复。验收不通过时按关键规则 10 处理。

### 关键规则

1. 不臆想需求，有依据才行动；不确定时停止并汇报
2. UPDATE.MD 是版本计划的事实来源，默认只开发下一版本
3. 子 Thread 必须定义明确的任务边界（含修改范围、影响范围、测试目标、测试边界）
4. 子 Agent 完成后必须提供标准化报告
5. 用户反馈的 BUG 和未完成内容，必须分配给子 Thread 处理
6. 测试唯一责任制：子 Agent 自测 = 单元测试；子 Thread 主 Agent 无需重复检查，直接汇报主 Thread；主 Thread 统一安排质量验证等后续测试；各级验收复用结论不重跑
7. 开始多 Thread 协作时创建 `.m-work-flow` 目录并加入 `.gitignore`
8. Thread 创建降级链：`create_thread` 失败 → `fork_thread` → 子 Agent 模式；降级后任务边界和报告格式不变，必须记录实际方式和降级原因
9. 前后端一致性测试：修改后端业务逻辑时必须验证前端调用和展示，修改前端交互时必须验证后端接口响应；测试报告中必须单独列出跨端验证结论
10. 验收问题闭环：子 Thread 主 Agent 验收不通过 → 退回子 Agent 修复；主 Thread 验收不通过 → 打回子 Thread 修复；同一执行者累计 3 轮验收不通过 → 升级创建新 Thread 独立修复

### 环境与部署规范

11. 环境权限：用户未说明正式环境时，一律视为开发环境或线上开发环境，权限全面打开，无需反复过问授权
12. 数据库清理：每次部署到线上开发环境时自动清理历史数据库，保证无脏数据污染，需初始化数据时清理后再次初始化
13. 数据重要性：开发环境中忽略数据重要性和安全性，直至用户提出要求时才需考虑



### Thread 管理要点

- 支持 `create_thread` 的 GUI 环境（Codex、opencode 等）：主 Thread 必须通过 `create_thread` 将子 Thread 创建为真实、可视化、用户可见的独立 Thread；不支持时才退化为内部子 Thread 并在 `.m-work-flow/threads/` 留痕
- `create_thread` 失败时的降级顺序：`create_thread` → `fork_thread`（分叉当前 Thread）→ 子 Agent 模式（在当前 Thread 内直接分配任务）；三种方式的任务边界、报告格式、验收流程不变
- 专项 Thread 按业务域命名（`thread-<领域>`），一个领域一个；已验收的进入"待复用"，同领域新问题优先重新激活，不新建
- 主 Thread 维护注册表 `.m-work-flow/threads/registry.md`（名称、领域、状态、复用次数、创建方式）

### 角色分工

- **流程统筹 Agent（主 Thread）**：整体方向、需求拆解、创建子 Thread、任务分配、跨子 Thread 交集测试、整体冒烟、验收和交付
- **子 Thread 主 Agent**：子 Thread 内部的任务规划、拆解、下发给子 Agent、汇总自测结果汇报主 Thread、处理多子 Agent 重叠
- **任务执行 Agent（子 Agent）**：执行分配的具体任务 + 单元测试自测，不越界
- **质量验证 Agent**：唯一的正式交互/功能测试层，由主 Thread 统一安排启动，验证修改范围内的功能并产出测试报告，供各级验收复用

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

### Skills 使用规范

项目实践必须使用指定的 Skill 工具，不得绕过或使用替代方案：

1. **浏览器相关实践与测试**：必须使用 `agent-browser`
   - 适用场景：页面导航、表单填写、按钮点击、截图、数据抓取、Web 应用测试、浏览器自动化等一切需要程序化操控浏览器的任务
   - 未安装时访问 GitHub 主页获取安装说明：https://github.com/vercel-labs/agent-browser
   - 禁止使用 Playwright、Puppeteer 或手写 Selenium 脚本替代

2. **前端设计相关实践**：必须使用 `web-design-guidelines`
   - 适用场景：UI 设计、界面审查、组件实现、响应式布局、无障碍、样式系统、交互设计等一切前端设计与编码任务
   - 未安装时从 `vercel-labs/agent-skills` 获取：https://github.com/vercel-labs/agent-skills
   - 禁止凭主观经验跳过设计规范直接编码

3. **工具优先原则**：当上述 Skill 已安装时，相关任务必须优先调用对应 Skill，不得手动重复其流程；Skill 未安装时必须先完成安装再开始任务，不得以"未安装"为由跳过
