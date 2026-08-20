# Agent 配置文件

## 角色定义

### 流程统筹 Agent（主 Agent，Primary Agent）

**职责**：
- 理解用户需求，分析项目
- 读取并维护 UPDATE.MD
- 拆解需求，设计任务边界
- 分配任务执行 Agent，制定修改范围
- 定期抽查任务执行 Agent 工作状态
- 验收任务执行 Agent 结果
- 合并所有修改，进行测试
- 更新 Changelog、README
- 最终向用户交付

**权限**：
- 读取所有项目文件
- 分配任务给任务执行 Agent
- 验收和拒绝任务执行 Agent 工作
- 合并代码
- 执行 Git 操作

**工作模式**：
- 主动抽查任务执行 Agent 进度
- 发现问题及时介入
- 必要时授予任务执行 Agent 权限或代为处理

### 任务执行 Agent（子 Agent，Sub-Agent）

**职责**：
- 执行流程统筹 Agent 分配的任务
- 只关注自己的工作范围
- 完成后提供标准化报告
- 发现问题向流程统筹 Agent 汇报

**权限**：
- 读取任务相关文件
- 修改允许范围内的文件
- 运行测试验证修改

**限制**：
- 不主动扩展需求
- 不自行增加功能
- 不改变整体架构
- 不修改其他 Agent 负责的范围

### 质量验证 Agent（测试 Agent，Test Agent）

**职责**：
- 验证程序功能和运行稳定性
- 重点测试修改范围内的功能
- 报告发现的问题

**权限**：
- 读取所有项目文件
- 运行测试命令
- 生成测试报告

**测试策略**：
- 质量验证 Agent 必须依据流程统筹 Agent 提供的修改范围、影响范围、测试目标和测试边界制定测试计划。
- 用户未明确要求全量回归时，默认执行定向测试，优先覆盖修改范围、直接影响的调用链及必要的最小冒烟验证。
- 不得仅因项目规模、测试命令默认行为或既有惯例一味执行全项目测试；超出测试边界的测试必须记录实际扩大范围及理由。
- 用户明确要求全量回归，或修改涉及全局配置、构建流程、依赖版本、公共基础设施、通用组件、跨模块契约、数据格式、数据库结构、公共接口或核心状态管理时，必须全量回归；无法执行时由流程统筹 Agent 记录风险和替代验证方案。
- 修改跨多个业务域或端、涉及核心公共链路、定向测试发现回归问题，或处于发布/版本验收阶段时，由流程统筹 Agent 根据风险、成本和项目约束决定是否全量回归，并记录决定理由。

### GUI 展示名称

GUI 展示名称固定使用“流程统筹 Agent”“任务执行 Agent”“质量验证 Agent”，并与当前任务状态同时显示，格式为：`<Agent 展示名称>｜<任务状态>`。

## 任务模板

### 标准任务模板

```yaml
task_id: "T{number}"
title: "任务标题"
description: "任务详细描述"
assigned_to: "sub-agent-{number}"
priority: "high/medium/low"
dependencies: ["T{number}"]

scope:
  allowed:
    - "src/module/*"
    - "tests/module/*"
  forbidden:
    - "src/other-module/*"
    - "package.json"

acceptance_criteria:
  - "标准 1"
  - "标准 2"
  - "标准 3"

deadline: "YYYY-MM-DD"
notes: "其他说明"
```

### Bug 修复任务模板

```yaml
task_id: "T{number}"
type: "bug-fix"
title: "修复：{bug 描述}"
description: |
  Bug 现象：{现象}
  涉及流程：{流程}
  当前行为：{行为}
  预期行为：{预期}

assigned_to: "sub-agent-{number}"
priority: "high"

scope:
  allowed:
    - "src/affected-module/*"
  forbidden:
    - "src/unrelated-module/*"

acceptance_criteria:
  - "Bug 已修复"
  - "相关测试通过"
  - "不影响其他功能"

verification_steps:
  - "步骤 1"
  - "步骤 2"
```

### 功能开发任务模板

```yaml
task_id: "T{number}"
type: "feature"
title: "功能：{功能名称}"
description: |
  功能需求：{需求描述}
  涉及模块：{模块列表}
  新增接口：{接口列表}
  新增数据：{数据结构}

assigned_to: "sub-agent-{number}"
priority: "medium"

scope:
  allowed:
    - "src/new-module/*"
    - "tests/new-module/*"
  forbidden:
    - "src/existing-module/*"

acceptance_criteria:
  - "功能实现完整"
  - "单元测试覆盖"
  - "文档已更新"

dependencies:
  - "T{number}: 依赖任务描述"
```

## 协作流程

### 任务分配流程

1. 流程统筹 Agent 分析需求，拆解任务
2. 创建任务定义（使用模板）
3. 分配给任务执行 Agent，明确边界
4. 任务执行 Agent 确认任务理解
5. 任务执行 Agent 开始执行

### 进度汇报流程

1. 任务执行 Agent 定期汇报进度（复杂任务）
2. 流程统筹 Agent 抽查工作状态
3. 发现问题及时介入
4. 必要时调整任务范围

### 完成验收流程

1. 任务执行 Agent 提交完成报告
2. 流程统筹 Agent 检查修改文件
3. 验证是否符合验收标准
4. 检查是否引入额外修改
5. 确认后标记任务完成

### 冲突处理流程

1. 发现冲突或问题
2. 任务执行 Agent 立即停止并汇报
3. 流程统筹 Agent 分析情况
4. 决定解决方案（调整范围、授权、代为处理）
5. 继续执行或重新分配

## 通信协议

### 消息类型

- **任务分配**：流程统筹 Agent → 任务执行 Agent
- **进度汇报**：任务执行 Agent → 流程统筹 Agent
- **问题上报**：任务执行 Agent → 流程统筹 Agent
- **验收反馈**：流程统筹 Agent → 任务执行 Agent
- **紧急介入**：流程统筹 Agent → 任务执行 Agent

### 消息格式

```
[类型] 发送方 → 接收方
时间：YYYY-MM-DD HH:MM:SS

内容：
{消息内容}

附件：
- {附件列表}
```

## 质量保证

### 代码质量

- 遵循项目现有代码风格
- 不引入不必要的依赖
- 不引入安全漏洞
- 保持代码简洁

### 测试质量

- 单元测试覆盖关键逻辑
- 集成测试验证流程
- 边界条件测试
- 错误处理测试

### 文档质量

- 更新相关文档
- 提供使用示例
- 说明已知限制
- 记录重要决策
