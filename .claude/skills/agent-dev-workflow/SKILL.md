---
name: agent-dev-workflow
description: |
  Agent 软件开发工作流程规范。适用于多 Session + 多 Agent 协作开发场景，包括需求分析、任务拆解、版本管理、代码验收和交付。
  触发条件：
  - 用户要求按 AGENTS.md 规范进行开发
  - 需要进行多 Session / 多 Agent 协作开发
  - 需要版本规划和管理（UPDATE.MD）
  - 需要任务拆解和分工
  - 需要子 Agent 验收流程
  - 用户提到"Agent 工作流"、"版本规划"、"任务拆解"、"Session"等关键词
---

# Agent 软件开发工作流程规范

## 核心原则

1. **先理解，再调研** - 需求优先，不臆想
2. **先拆解，再分工** - 明确边界，职责隔离
3. **先执行，再汇报** - 标准化报告
4. **先验收，再合并** - 流程统筹 Agent 负责最终质量
5. **先验证，再交付** - 测试唯一责任，结论逐级复用

## Session 层级架构

### 层级结构

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

### 子 Session 特性

子 Session 是**任务执行者**，不是任务规划者：
- 只负责落地主 Session 拆解下来可执行的任务
- 不会像 Agent 有多个角色区分
- 内部由子 Session 主 Agent 规划具体执行细节
- 对主 Session 负责，汇报执行结果

### Session 形态与管理

Session 分为两种形态：

- **可视化 Session**：在 Codex、opencode 等支持多会话的 GUI 环境中，子 Session 必须创建为真实、用户可见的独立会话
- **内部子会话**：仅当环境不支持多会话时使用，且必须在 `.m-work-flow/sessions/` 中完整留痕

管理规则：

- 专项 Session 原则：按业务域建立专项 Session（`session-<领域名>`），一个领域一个
- 复用优先：已完成验收的 Session 进入"待复用"并保留上下文；同领域新问题优先重新激活，不新建
- 生命周期：创建 → 执行中 → 已验收 → 待复用 → （重新激活）→ 关闭
- 注册表：`.m-work-flow/sessions/registry.md` 记录所有 Session 的名称、领域、状态、复用次数

详细规范 → [references/session-management.md](references/session-management.md)

## 开发流程

```
需求理解 → 项目分析 → GitHub 调研 → 需求拆解 → 创建子 Session → 子 Session 内部规划 → 子 Agent 执行 → 定向测试 → 验收 → 合并 → 交付
```

## 关键规则

### 需求管理
- 不臆想需求，有依据才行动
- 用户需求 > 已有代码 > Agent 假设
- 遇到不确定，停止并汇报

### 版本管理
- `UPDATE.MD` 是版本计划的事实来源
- 默认只开发下一版本
- 版本完成后更新当前版本号并保留历史

### 测试去重

- 测试唯一责任制：同一修改范围的完整测试只执行一次
- 任务执行 Agent 只做最小自测；质量验证 Agent 是唯一正式定向测试层
- 验收层只审查测试证据，不重跑已覆盖的测试
- 主 Session 只测跨子 Session 交集和整体启动冒烟，复用子 Session 测试结论

### 用户反馈修复
- 用户校验后的 BUG 修复和功能完善，必须分配给任务执行 Agent 执行
- 流程统筹 Agent 不直接修复代码，只负责分析、拆解和分配
- 修复任务必须遵循标准任务边界格式
- 修复完成后必须经过流程统筹 Agent 验收

### 角色分工

#### 主 Session 层级
- **流程统筹 Agent（主 Session Agent）**：负责整体方向、需求拆解、创建子 Session、任务分配、验收和交付

#### 子 Session 层级
- **子 Session 主 Agent**：负责子 Session 内部的任务规划、拆解和下发给子 Agent
- **任务执行 Agent（子 Agent）**：负责执行子 Session 主 Agent 分配的具体任务，不越界
- **质量验证 Agent（测试 Agent）**：唯一的正式定向测试层，验证修改范围内的功能并产出测试报告，供各级验收复用

#### GUI 展示标识

- 四类角色的 GUI 展示名称固定使用"流程统筹 Agent""子 Session 主 Agent""任务执行 Agent""质量验证 Agent"，不得仅使用"主 Agent""子 Agent"或"测试 Agent"作为展示名称。
- 展示名称必须与当前任务状态同时显示，格式为：`<角色展示名称>｜<任务状态>`，例如：`任务执行 Agent｜工作中`。
- 子 Session 级别需展示 Session 标识，格式为：`Session:<Session名称>｜<角色展示名称>｜<任务状态>`

### 任务边界

#### 主 Session 任务边界格式
主 Session 在拆解任务时必须定义：
- 任务描述
- 子 Session 划分依据
- 子 Session 依赖关系
- 验收标准
- 修改范围：实际允许变更的模块、文件或功能
- 影响范围：直接调用链、依赖模块、资源、数据或用户流程
- 测试目标：需要验证的行为和风险
- 测试边界：本次默认不覆盖的范围；如需全量回归需明确说明

#### 子 Session 任务边界格式
子 Session 主 Agent 在分配任务时必须定义：
- 任务描述
- 允许修改范围
- 禁止修改范围
- 验收标准
- 修改范围：实际允许变更的模块、文件或功能
- 影响范围：直接调用链、依赖模块、资源、数据或用户流程
- 测试目标：需要验证的行为和风险
- 测试边界：本次默认不覆盖的范围；如需全量回归需明确说明

子 Session 主 Agent 分配任务时必须填写上述四项测试相关信息。质量验证 Agent 应以分配的修改范围和影响范围为依据制定测试，不得在用户未明确要求全量回归时默认执行全项目测试；扩大测试范围时必须在测试报告中记录扩大范围及理由。

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

根据需要查阅以下参考文档：

- **开发流程详情** → [references/development-workflow.md](references/development-workflow.md)
- **版本管理规范** → [references/version-management.md](references/version-management.md)
- **Session 管理规范** → [references/session-management.md](references/session-management.md)
- **Agent 角色职责** → [references/agent-roles.md](references/agent-roles.md)
- **测试规范** → [references/testing.md](references/testing.md)
- **交付规范** → [references/delivery.md](references/delivery.md)

## Token 优化

- 局部分析，避免全项目扫描
- 子 Session / 子 Agent 通过上下文传递信息，不重复读取
- 测试聚焦修改范围，不盲目全覆盖
- 同一修改范围只完整测试一次，验收层复用上游测试结论，不重跑
- 复用已有代码和开源方案
- 同领域任务优先复用已有专项 Session，沿用其上下文和测试结论
- 主 Session 创建 `.m-work-flow` 目录存放子 Session 和子 Agent 需要阅读的内容
- 避免子 Session / 子 Agent 反复重读项目文件

## 工作流目录规范

### .m-work-flow 目录

流程统筹 Agent 在开始多 Session / 多 Agent 协作时，必须在项目根目录创建 `.m-work-flow` 隐藏目录，用于：

1. **存放上下文内容**
   - 项目关键文件摘要
   - 代码结构分析
   - 相关配置信息
   - 需求文档摘要
   - 避免子 Session / 子 Agent 重复读取项目文件

2. **保存 Planner 文件**
   - 主 Session 任务拆解计划
   - 子 Session 划分方案
   - 子 Agent 分配方案
   - 时间预估和进度跟踪

3. **通信记录**
   - 主 Session 与子 Session 的通信内容
   - 子 Session 与子 Agent 的通信内容
   - 子 Agent 完成报告
   - 子 Session 完成报告
   - 验收结果

### 目录结构

```
.m-work-flow/
├── context/                    # 上下文内容
│   ├── project-summary.md
│   ├── code-structure.md
│   └── requirements.md
├── planner/                    # Planner 文件
│   ├── session-breakdown.md    # 主 Session 任务拆解
│   ├── session-assignment.md   # 子 Session 划分方案
│   └── agent-assignment.md     # 子 Agent 分配方案
├── sessions/                   # 子 Session 工作目录
│   ├── registry.md             # Session 注册表（名称、领域、状态、复用次数）
│   ├── session-1/              # 子 Session 1
│   │   ├── tasks/              # 任务分配
│   │   ├── reports/            # 完成报告
│   │   └── communication/      # 通信记录
│   └── session-2/              # 子 Session 2
│       ├── tasks/
│       ├── reports/
│       └── communication/
├── communication/              # 主 Session 通信记录
│   ├── main-session/
│   └── final-reports/
└── timeline/                   # 时间跟踪
    └── timeout-monitor.md
```

### Git 排除规则

如果项目存在 Git 环境，必须将 `.m-work-flow` 添加到 `.gitignore` 文件中：

```gitignore
# Agent 工作流目录
.m-work-flow/
```

**原因**：
- `.m-work-flow` 是临时工作目录，包含主 Session、子 Session 和子 Agent 的通信内容
- 不应被提交到版本控制系统
- 避免污染项目历史记录

## 超时问询机制

### 多层级问询架构

超时问询机制分为两个层级：

#### 主 Session → 子 Session 问询
- 主 Session 定期检查子 Session 工作状态
- 超时时主 Session 主动向子 Session 问询
- 问询内容：整体进度、阻塞问题、是否需要支持

#### 子 Session → 子 Agent 问询
- 子 Session 主 Agent 定期检查子 Agent 工作状态
- 超时时子 Session 主 Agent 主动向子 Agent 问询
- 问询内容：当前工作进度、遇到的困难、预计剩余时间

### 时间预估要求

子 Session 主 Agent 在分配子 Agent 任务时，必须提供完成时间预估：

1. **预估内容**
   - 任务预计完成时间
   - 检查时间点
   - 超时阈值

2. **预估格式**
   ```
   任务：<描述>
   预计完成时间：<时间>
   检查时间点：<时间>
   超时阈值：<时间>
   ```

### 超时检查流程

1. **定时检查**
    - 子 Session 主 Agent 定期检查子 Agent 工作状态
   - 检查时间同步到用户界面
   - 记录检查结果

2. **超时问询**
    - 当超过预估时间未收到子 Agent 通知时
    - 子 Session 主 Agent 主动向子 Agent 问询
   - 问询内容包括：
     - 当前工作进度
     - 遇到的困难或阻塞
     - 是否需要额外支持
     - 预计剩余时间

3. **用户界面同步**
   - 检查时间同步到用户界面
    - 显示子 Agent 工作状态
   - 显示超时预警信息

### 问询格式

```
问询时间：<当前时间>
层级：<主 Session / 子 Session>
角色：<稳定职能名称>
任务：<任务描述>
预估完成时间：<预估时间>
当前状态：<状态描述>
问询原因：<超时/主动检查>
```

## 语言规范

- 用户端 UI 文案：默认中文（除非用户指定其他语言）
- 代码、变量名、日志：可保留英文
- 技术文档：跟随项目原有语言
