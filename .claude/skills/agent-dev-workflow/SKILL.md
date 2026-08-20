---
name: agent-dev-workflow
description: |
  Agent 软件开发工作流程规范。适用于多 Agent 协作开发场景，包括需求分析、任务拆解、版本管理、代码验收和交付。
  触发条件：
  - 用户要求按 AGENTS.md 规范进行开发
  - 需要进行多 Agent 协作开发
  - 需要版本规划和管理（UPDATE.MD）
  - 需要任务拆解和分工
  - 需要子 Agent 验收流程
  - 用户提到"Agent 工作流"、"版本规划"、"任务拆解"等关键词
---

# Agent 软件开发工作流程规范

## 核心原则

1. **先理解，再调研** - 需求优先，不臆想
2. **先拆解，再分工** - 明确边界，职责隔离
3. **先执行，再汇报** - 标准化报告
4. **先验收，再合并** - 主 Agent 负责最终质量
5. **先验证，再交付** - 测试驱动

## 开发流程

```
需求理解 → 项目分析 → GitHub 调研 → 需求拆解 → Agent 分工 → 子任务执行 → 验收 → 测试 → 合并 → 交付
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

### 用户反馈修复
- 用户校验后的 BUG 修复和功能完善，必须分配给子 Agent 执行
- 主 Agent 不直接修复代码，只负责分析、拆解和分配
- 修复任务必须遵循标准任务边界格式
- 修复完成后必须经过主 Agent 验收

### Agent 分工
- 主 Agent：整体方向、任务分配、验收、交付
- 子 Agent：执行分配的任务，不越界
- 测试 Agent：验证修改范围内的功能

### 任务边界
每个子任务必须定义：
- 任务描述
- 允许修改范围
- 禁止修改范围
- 验收标准

### 完成报告格式
```
任务：<描述>
修改文件：<列表>
完成内容：<列表>
测试结果：<内容>
发现的问题：<内容>
额外修改：<有/无>
需要主 Agent 注意：<内容>
```

## 详细规范

根据需要查阅以下参考文档：

- **开发流程详情** → [references/development-workflow.md](references/development-workflow.md)
- **版本管理规范** → [references/version-management.md](references/version-management.md)
- **Agent 角色职责** → [references/agent-roles.md](references/agent-roles.md)
- **测试规范** → [references/testing.md](references/testing.md)
- **交付规范** → [references/delivery.md](references/delivery.md)

## Token 优化

- 局部分析，避免全项目扫描
- 子 Agent 通过上下文传递信息，不重复读取
- 测试聚焦修改范围，不盲目全覆盖
- 复用已有代码和开源方案
- 主 Agent 创建 `.m-work-flow` 目录存放子 Agent 需要阅读的内容
- 避免子 Agent 反复重读项目文件

## 工作流目录规范

### .m-work-flow 目录

主 Agent 在开始多 Agent 协作时，必须在项目根目录创建 `.m-work-flow` 隐藏目录，用于：

1. **存放子 Agent 需要阅读的内容**
   - 项目关键文件摘要
   - 代码结构分析
   - 相关配置信息
   - 需求文档摘要
   - 避免子 Agent 重复读取项目文件

2. **保存 Planner 文件**
   - 任务拆解计划
   - 子 Agent 分配方案
   - 时间预估和进度跟踪

3. **通信记录**
   - 主 Agent 与子 Agent 的通信内容
   - 子 Agent 完成报告
   - 验收结果

### 目录结构

```
.m-work-flow/
├── context/           # 子 Agent 需要阅读的上下文内容
│   ├── project-summary.md
│   ├── code-structure.md
│   └── requirements.md
├── planner/           # Planner 文件
│   ├── task-breakdown.md
│   └── agent-assignment.md
├── communication/     # 通信记录
│   ├── subagent-tasks/
│   └── subagent-reports/
└── timeline/          # 时间跟踪
    └── timeout-monitor.md
```

### Git 排除规则

如果项目存在 Git 环境，必须将 `.m-work-flow` 添加到 `.gitignore` 文件中：

```gitignore
# Agent 工作流目录
.m-work-flow/
```

**原因**：
- `.m-work-flow` 是临时工作目录，包含主 Agent 和子 Agent 的通信内容
- 不应被提交到版本控制系统
- 避免污染项目历史记录

## 超时问询机制

### 时间预估要求

主 Agent 在分配子 Agent 任务时，必须提供完成时间预估：

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
   - 主 Agent 定期检查子 Agent 工作状态
   - 检查时间同步到用户界面
   - 记录检查结果

2. **超时问询**
   - 当超过预估时间未收到子 Agent 通知时
   - 主 Agent 主动向子 Agent 问询
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
子 Agent：<子 Agent 名称>
任务：<任务描述>
预估完成时间：<预估时间>
当前状态：<状态描述>
问询原因：<超时/主动检查>
```

## 语言规范

- 用户端 UI 文案：默认中文（除非用户指定其他语言）
- 代码、变量名、日志：可保留英文
- 技术文档：跟随项目原有语言
