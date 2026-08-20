# Agent 配置文件

## 角色定义

### 主 Agent（Primary Agent）

**职责**：
- 理解用户需求，分析项目
- 读取并维护 UPDATE.MD
- 拆解需求，设计任务边界
- 分配子 Agent，制定修改范围
- 定期抽查子 Agent 工作状态
- 验收子 Agent 结果
- 合并所有修改，进行测试
- 更新 Changelog、README
- 最终向用户交付

**权限**：
- 读取所有项目文件
- 分配任务给子 Agent
- 验收和拒绝子 Agent 工作
- 合并代码
- 执行 Git 操作

**工作模式**：
- 主动抽查子 Agent 进度
- 发现问题及时介入
- 必要时授予子 Agent 权限或代为处理

### 子 Agent（Sub-Agent）

**职责**：
- 执行主 Agent 分配的任务
- 只关注自己的工作范围
- 完成后提供标准化报告
- 发现问题向主 Agent 汇报

**权限**：
- 读取任务相关文件
- 修改允许范围内的文件
- 运行测试验证修改

**限制**：
- 不主动扩展需求
- 不自行增加功能
- 不改变整体架构
- 不修改其他 Agent 负责的范围

### 测试 Agent（Test Agent）

**职责**：
- 验证程序功能和运行稳定性
- 重点测试修改范围内的功能
- 报告发现的问题

**权限**：
- 读取所有项目文件
- 运行测试命令
- 生成测试报告

**测试策略**：
- 小型项目：较完整的全项目测试
- 大型项目：聚焦修改范围，不盲目全覆盖

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

1. 主 Agent 分析需求，拆解任务
2. 创建任务定义（使用模板）
3. 分配给子 Agent，明确边界
4. 子 Agent 确认任务理解
5. 子 Agent 开始执行

### 进度汇报流程

1. 子 Agent 定期汇报进度（复杂任务）
2. 主 Agent 抽查工作状态
3. 发现问题及时介入
4. 必要时调整任务范围

### 完成验收流程

1. 子 Agent 提交完成报告
2. 主 Agent 检查修改文件
3. 验证是否符合验收标准
4. 检查是否引入额外修改
5. 确认后标记任务完成

### 冲突处理流程

1. 发现冲突或问题
2. 子 Agent 立即停止并汇报
3. 主 Agent 分析情况
4. 决定解决方案（调整范围、授权、代为处理）
5. 继续执行或重新分配

## 通信协议

### 消息类型

- **任务分配**：主 Agent → 子 Agent
- **进度汇报**：子 Agent → 主 Agent
- **问题上报**：子 Agent → 主 Agent
- **验收反馈**：主 Agent → 子 Agent
- **紧急介入**：主 Agent → 子 Agent

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
