# Agent 快速参考卡

## 主 Agent 检查清单

### 任务开始前

- [ ] 理解用户需求
- [ ] 分析项目结构
- [ ] 读取 UPDATE.MD（如涉及版本）
- [ ] 拆解需求为最小单元
- [ ] 定义每个任务的边界

### 任务分配时

- [ ] 使用任务模板创建任务
- [ ] 明确允许修改范围
- [ ] 明确禁止修改范围
- [ ] 定义验收标准
- [ ] 说明依赖关系

### 执行过程中

- [ ] 定期抽查子 Agent 进度
- [ ] 发现问题及时介入
- [ ] 必要时调整任务范围
- [ ] 授权或代为处理阻塞问题

### 任务完成后

- [ ] 检查完成报告
- [ ] 验证修改文件
- [ ] 确认符合验收标准
- [ ] 检查是否引入额外修改
- [ ] 标记任务完成

## 子 Agent 检查清单

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
- 发现问题向主 Agent 汇报

### 完成报告

- 必须使用标准化格式
- 真实反映实际修改
- 不隐瞒额外修改

## 紧急情况处理

### 子 Agent 阻塞

1. 子 Agent 立即汇报
2. 主 Agent 分析原因
3. 授权或代为处理
4. 继续执行或重新分配

### 发现冲突

1. 子 Agent 立即停止
2. 汇报冲突详情
3. 主 Agent 协调解决
4. 调整任务范围

### 需求变更

1. 主 Agent 与用户确认
2. 评估影响范围
3. 调整任务计划
4. 通知相关子 Agent

## 参考文档

- **完整规范**：`.claude/skills/agent-dev-workflow/SKILL.md`
- **Agent 配置**：`.claude/agents/AGENT_CONFIG.md`
- **任务模板**：`.claude/agents/templates/TASK_TEMPLATES.md`
- **开发流程**：`.claude/skills/agent-dev-workflow/references/development-workflow.md`
- **版本管理**：`.claude/skills/agent-dev-workflow/references/version-management.md`
- **Agent 角色**：`.claude/skills/agent-dev-workflow/references/agent-roles.md`
- **测试规范**：`.claude/skills/agent-dev-workflow/references/testing.md`
- **交付规范**：`.claude/skills/agent-dev-workflow/references/delivery.md`
