---
name: agent-dev-workflow
description: |
  Agent 软件开发工作流程规范。适用于多 Thread + 多 Agent 协作开发场景，包括需求分析、任务拆解、版本管理、代码验收和交付。
  触发条件：
  - 用户要求按 AGENTS.md 规范进行开发
  - 需要进行多 Thread / 多 Agent 协作开发
  - 需要版本规划和管理（UPDATE.MD）
  - 需要任务拆解和分工
  - 需要子 Agent 验收流程
  - 用户提到"Agent 工作流"、"版本规划"、"任务拆解"、"Thread"等关键词
---

# Agent 软件开发工作流程规范

## 核心原则

1. **先理解，再调研** - 需求优先，不臆想
2. **先拆解，再分工** - 明确边界，职责隔离
3. **先执行，再汇报** - 标准化报告
4. **先验收，再合并** - 流程统筹 Agent 负责最终质量
5. **先验证，再交付** - 测试唯一责任，结论逐级复用

## Thread 层级架构

主 Thread（流程统筹 Agent）负责整体需求拆解、任务规划、最终验收和交付；通过 `create_thread` 命令将可执行任务创建为独立的子 Thread。每个子 Thread 内部由子 Thread 主 Agent 规划，并下发任务给多个子 Agent 执行。

**Thread 与 Agent 的关键区分**：

- Thread 是任务执行的容器：开启新的执行容器必须由主 Thread 调用 `create_thread` 创建
- Agent 是任务执行者：不通过 `create_thread` 创建，而是在所属 Thread 内部由上级 Agent 下发任务形成
- 一个子 Thread 内可包含多个子 Agent；不为每个子 Agent 单独创建 Thread

详细定义、形态判定、专项复用和注册表规范 → [references/thread-management.md](references/thread-management.md)

## 开发主流程

```
需求理解 → 项目分析 → GitHub 调研 → 需求拆解 → 创建子 Thread → 子 Thread 内部规划 → 子 Agent 执行 → 子 Agent 自测 → 质量验证 → 验收 → 合并 → 交付
```

## 角色分工

| 角色 | 所处层级 | 一句话职责 |
|------|---------|-----------|
| 流程统筹 Agent | 主 Thread | 整体方向、需求拆解、`create_thread`（失败时按降级链处理）创建子 Thread、跨子 Thread 交集测试、整体冒烟、验收和交付 |
| 子 Thread 主 Agent | 子 Thread | 子 Thread 内部的任务规划、拆解、下发给子 Agent、判断测试启动时机、处理多子 Agent 重叠 |
| 任务执行 Agent（子 Agent） | 子 Thread 内 | 执行分配的具体任务 + 单元测试自测，不越界 |
| 质量验证 Agent（测试 Agent） | 子 Thread 内 | 唯一的正式交互/功能测试层，所有编写性工作完成后统一启动，产出测试报告供各级验收复用 |

角色职责详解、GUI 展示标识、任务边界格式和各类报告模板 → [references/agent-roles.md](references/agent-roles.md)

## 关键规则索引

- **需求管理**：不臆想需求，有依据才行动；不确定时停止并汇报 → 细则见 [references/development-workflow.md](references/development-workflow.md)
- **版本管理**：`UPDATE.MD` 是版本计划的事实来源，默认只开发下一版本 → 细则见 [references/version-management.md](references/version-management.md)
- **测试去重**：子 Agent 自测 = 单元测试；质量验证 Agent = 交互/功能测试，所有编写性工作完成后统一启动；各级验收复用结论不重跑 → 细则见 [references/testing.md](references/testing.md)
- **用户反馈修复**：BUG 修复必须走"分析 → 拆解 → 分配子 Thread"流程，流程统筹 Agent 不直接修复 → 细则见 [references/development-workflow.md](references/development-workflow.md)
- **Thread 管理**：支持 `create_thread` 的环境必须创建真实可视化的子 Thread；专项命名、复用优先、注册表维护 → 细则见 [references/thread-management.md](references/thread-management.md)
- **协作目录**：开始协作时创建 `.m-work-flow` 目录存放上下文、Planner 文件和通信记录，并加入 `.gitignore` → 目录结构与用法见 [references/development-workflow.md](references/development-workflow.md)
- **交付**：合并、Changelog、Git 提醒、README 和语言规范 → 见 [references/delivery.md](references/delivery.md)

## 详细规范

- **开发流程详情**（含超时问询机制、`.m-work-flow` 目录） → [references/development-workflow.md](references/development-workflow.md)
- **版本管理规范** → [references/version-management.md](references/version-management.md)
- **Thread 管理规范**（含 `create_thread` 使用规则） → [references/thread-management.md](references/thread-management.md)
- **Agent 角色职责**（含任务边界格式、完成报告格式、验收流程） → [references/agent-roles.md](references/agent-roles.md)
- **测试规范** → [references/testing.md](references/testing.md)
- **交付规范** → [references/delivery.md](references/delivery.md)
