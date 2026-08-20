# Agent 快速参考卡

## 流程统筹 Agent（主 Agent）检查清单

### 任务开始前

- [ ] 理解用户需求
- [ ] 分析项目结构
- [ ] 读取 UPDATE.MD（如涉及版本）
- [ ] 拆解需求为最小单元
- [ ] 定义每个任务的边界

### 任务分配时

- [ ] 使用任务模板创建任务
- [ ] 明确修改范围（允许变更的模块、文件或功能）
- [ ] 明确禁止修改范围
- [ ] 定义验收标准
- [ ] 说明依赖关系
- [ ] 明确影响范围
- [ ] 明确测试目标
- [ ] 明确测试边界

### 执行过程中

- [ ] 定期抽查任务执行 Agent 进度
- [ ] 发现问题及时介入
- [ ] 必要时调整任务范围
- [ ] 授权或代为处理阻塞问题

### 任务完成后

- [ ] 检查完成报告
- [ ] 验证修改文件
- [ ] 确认符合验收标准
- [ ] 检查是否引入额外修改
- [ ] 标记任务完成

## 任务执行 Agent（子 Agent）检查清单

### 接收任务时

- [ ] 理解任务描述
- [ ] 确认修改范围
- [ ] 明确验收标准
- [ ] 识别依赖关系
- [ ] 如有疑问，立即询问

### 执行过程中

- [ ] 只修改允许范围内的文件
- [ ] 不扩展需求
- [ ] 不引入额外功能
- [ ] 遇到问题立即汇报
- [ ] 定期汇报进度（复杂任务）

### 完成任务时

- [ ] 使用完成报告模板
- [ ] 列出所有修改文件
- [ ] 说明测试结果
- [ ] 报告发现的问题
- [ ] 说明额外修改（如有）
- [ ] 提醒需要注意的事项

## 常用命令

### 查看任务列表

```bash
# 查看所有任务
ls .claude/tasks/

# 查看特定任务
cat .claude/tasks/T1/task.md
```

### 创建任务

```bash
# 创建任务目录
mkdir -p .claude/tasks/T{number}

# 从模板创建任务
cp .claude/templates/standard-task.md .claude/tasks/T{number}/task.md
```

### 更新任务状态

```bash
# 任务进行中
echo "in_progress" > .claude/tasks/T{number}/status

# 任务完成
echo "completed" > .claude/tasks/T{number}/status

# 任务阻塞
echo "blocked" > .claude/tasks/T{number}/status
```

## 关键规则速查

### 测试范围

- 质量验证 Agent 依据流程统筹 Agent 提供的修改范围、影响范围、测试目标和测试边界制定测试。
- 用户未明确要求全量回归时，默认执行定向测试，覆盖修改范围、直接影响链路和必要的最小冒烟验证。
- 不得仅因项目规模、测试命令默认行为或惯例一味执行全项目测试；超出测试边界时须记录扩大范围和理由。
- 用户明确要求全量回归，或修改涉及全局配置、构建流程、依赖版本、公共基础设施、通用组件、跨模块契约、数据格式、数据库结构、公共接口或核心状态管理时，必须全量回归；无法执行时由流程统筹 Agent 记录风险和替代验证方案。
- 跨多个业务域或端、涉及核心公共链路、定向测试发现回归问题，或发布前/版本验收时，由流程统筹 Agent 根据风险、成本和项目约束决定是否全量回归，并记录理由。

### 需求管理

- 不臆想需求
- 用户需求 > 已有代码 > Agent 假设
- 遇到不确定，停止并汇报

### 版本管理

- UPDATE.MD 是版本计划的事实来源
- 默认只开发下一版本
- 版本完成后更新当前版本号

### 修改边界

- 只修改允许范围内的文件
- 不扩展需求，不引入额外功能
- 发现问题向流程统筹 Agent（主 Agent）汇报

### 完成报告

- 必须使用标准化格式
- 真实反映实际修改
- 不隐瞒额外修改

## 紧急情况处理

### 任务执行 Agent 阻塞

1. 任务执行 Agent 立即汇报
2. 流程统筹 Agent 分析原因
3. 授权或代为处理
4. 继续执行或重新分配

### 发现冲突

1. 任务执行 Agent 立即停止
2. 汇报冲突详情
3. 流程统筹 Agent 协调解决
4. 调整任务范围

### 需求变更

1. 流程统筹 Agent 与用户确认
2. 评估影响范围
3. 调整任务计划
4. 通知相关任务执行 Agent

## 参考文档

- **完整规范**：`.claude/skills/agent-dev-workflow/SKILL.md`
- **Agent 配置**：`.claude/agents/AGENT_CONFIG.md`
- **任务模板**：`.claude/agents/templates/TASK_TEMPLATES.md`
- **开发流程**：`.claude/skills/agent-dev-workflow/references/development-workflow.md`
- **版本管理**：`.claude/skills/agent-dev-workflow/references/version-management.md`
- **Agent 角色**：`.claude/skills/agent-dev-workflow/references/agent-roles.md`
- **测试规范**：`.claude/skills/agent-dev-workflow/references/testing.md`
- **交付规范**：`.claude/skills/agent-dev-workflow/references/delivery.md`
