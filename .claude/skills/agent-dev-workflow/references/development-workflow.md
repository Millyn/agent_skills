# 开发流程详情

> 本文是以下内容的唯一权威来源：开发各阶段细则、防止 Agent 臆想、停止条件、Token 优化、`.m-work-flow` 目录规范、用户反馈修复流程、超时问询机制。
> Thread 创建规则见 [thread-management.md](thread-management.md)；任务边界与报告模板见 [agent-roles.md](agent-roles.md)。

## 流程总览

```
需求理解 → 项目分析 → GitHub 调研 → 需求拆解 → 创建子 Thread → 子 Thread 内部规划 → 子 Agent 执行 → 定向测试 → 验收 → 合并 → 交付
```

## 需求理解阶段

主 Thread（流程统筹 Agent）接收需求后，必须先理解：

1. 用户真正需要解决的问题
2. 当前项目已具备的功能
3. 用户要求新增、修改或删除的功能
4. 功能之间的依赖关系
5. 哪些内容属于/不属于本次需求

信息来源优先级：用户需求描述 > 当前项目代码 > 项目文档 > 配置文件 > 已有测试 > Git 历史 > GitHub 类似开源项目。

## GitHub 开源调研

- **目的**：学习已有实现、复用成熟设计；避免重新设计已存在的解决方案；减少 Token 消耗和错误实现概率
- **优先寻找**：功能高度相似的项目、相似技术栈项目、官方示例项目、社区成熟且长期维护的实现
- **注意事项**：开源项目只能作为学习和参考，不应直接复制实现；必须考虑当前项目技术栈和实际需求；不引入不需要的新依赖或复杂架构

## 需求拆解

### 主 Thread 拆解流程

将需求拆解为子 Thread，每个子 Thread 应满足：

- 有明确的任务目标
- 有独立的修改范围
- 有明确的验收标准
- 尽量减少与其他子 Thread 的依赖

创建子 Thread 前：

- 先查 `.m-work-flow/threads/registry.md`，存在同领域待复用 Thread 时优先重新激活，不新建（复用规则见 [thread-management.md](thread-management.md)）
- 在支持 `create_thread` 的环境中必须真实创建可视化 Thread，命名遵循 `thread-<领域名>`（形态判定与 `create_thread` 规则见 [thread-management.md](thread-management.md)）

### 子 Thread 内部拆解

子 Thread 主 Agent 接收任务后，将任务拆解为最小工作单元，每个单元应满足：

- 有明确的输入和输出
- 有明确的修改范围
- 有明确的验收标准
- 尽量减少与其他单元的文件和逻辑交叉

任务边界的标准格式见 [agent-roles.md](agent-roles.md)「任务边界格式」。

### 拆解示例

```
需求
├── 子 Thread A：后端功能
│   ├── 子 Agent 1：API 接口
│   └── 子 Agent 2：数据库修改
├── 子 Thread B：前端功能
│   ├── 子 Agent 3：页面开发
│   └── 子 Agent 4：资源加载
└── 子 Thread C：测试
    └── 子 Agent 5：功能测试
```

## 防止 Agent 臆想

**事实 > 推测，已有代码 > 个人假设，用户需求 > Agent 想法**

无法确定时的处理顺序：查看现有代码 → 项目文档 → 配置文件 → 相关测试 → Git 历史 → GitHub 类似实现。

禁止无依据创造：新业务规则、新功能、新数据结构、新接口、新依赖、新的用户需求。

## 停止条件

出现以下情况时立即停止并向上级汇报，等待决策。不要自行扩大需求、改变架构、修改其他模块或引入新依赖。

正确行为：**停止 → 汇报 → 等待上级决策**

- **子 Agent → 子 Thread 主 Agent**：用户需求存在歧义；需求与项目现有逻辑冲突；缺少完成任务必需的信息或依赖；现有架构无法直接支持需求；修改可能影响其他模块；其他子 Agent 已修改相关代码；无法确定行为应该如何实现
- **子 Thread 主 Agent → 主 Thread**：子 Thread 任务与整体需求冲突；子 Thread 之间存在无法解决的依赖；子 Agent 遇到阻塞且无法自行解决；需要主 Thread 决策的问题

## Token 与资源优化

优先采用：

- 局部代码分析和文件读取、局部测试
- 已有代码复用、GitHub 开源项目参考
- 子 Thread / 子 Agent 职责隔离
- 使用 UPDATE.MD 获取明确版本目标
- 主 Thread 创建 `.m-work-flow` 目录存放子 Thread 和子 Agent 需要阅读的内容，避免反复重读项目文件

避免：

- 重复读取同一份代码、重复分析已确定的需求
- 无意义扫描或测试整个大型项目
- 在用户未要求时进行性能测试
- 重复实现已存在的成熟功能
- 为了"可能有用"增加额外功能
- 子 Thread / 子 Agent 重复读取主 Thread 已整理好的内容

## .m-work-flow 目录规范

流程统筹 Agent 在开始多 Thread / 多 Agent 协作时，必须在项目根目录创建 `.m-work-flow` 隐藏目录，用于存放上下文内容、Planner 文件和通信记录，使各级通过上下文传递信息而不重复读取项目文件：

```
.m-work-flow/
├── context/                    # 上下文内容
│   ├── project-summary.md      # 项目关键文件摘要
│   ├── code-structure.md       # 代码结构分析
│   └── requirements.md         # 需求文档摘要
├── planner/                    # Planner 文件
│   ├── thread-breakdown.md     # 主 Thread 任务拆解计划
│   ├── thread-assignment.md    # 子 Thread 划分方案
│   └── agent-assignment.md     # 子 Agent 分配方案
├── threads/                    # 子 Thread 工作目录
│   ├── registry.md             # Thread 注册表（见 thread-management.md）
│   └── thread-1/               # 每个 Thread 含 tasks/、reports/、communication/
├── communication/              # 主 Thread 通信记录
│   ├── main-thread/
│   └── final-reports/
└── timeline/                   # 时间跟踪
    └── timeout-monitor.md
```

- **context/**：项目关键文件摘要、代码结构分析、相关配置信息、需求文档摘要——避免子 Thread / 子 Agent 重复读取项目文件
- **planner/**：主 Thread 任务拆解计划、子 Thread 划分方案、子 Agent 分配方案、时间预估和进度跟踪
- **communication/ 与 threads/\*/reports/**：主 Thread 与子 Thread 的通信记录、子 Agent 与子 Thread 的完成报告、验收结果

如果项目存在 Git 环境，必须将 `.m-work-flow/` 添加到 `.gitignore`：它是临时工作目录，包含各级通信内容，不应提交到版本控制系统，避免污染项目历史记录。

## 用户反馈修复流程

当用户校验结果并反馈 BUG 或未完成内容时，主 Thread 必须按规范流程处理，而不是直接修复：

```
用户反馈 → 问题分析 → 创建子 Thread → 子 Thread 内部规划 → 子 Agent 修复 → 验收 → 交付
```

- **主 Thread**：接收反馈并记录 → 分析问题原因和影响范围 → 通过 `create_thread` 将修复工作创建为独立子 Thread → 验收子 Thread 修复结果 → 向用户交付
- **子 Thread**：将修复任务按标准「子 Thread 任务定义格式」（见 [agent-roles.md](agent-roles.md)）拆解分配给子 Agent → 监督执行 → 验收 → 向主 Thread 汇报

修复任务的验收标准至少包含：原始问题是否已解决、是否引入新的问题、是否影响其他功能、是否符合项目规范。

禁止行为：主 Thread 直接修复代码；子 Thread 主 Agent 直接修复代码；跳过任务拆解直接执行；跳过验收直接交付；修复范围超出用户反馈。

### 示例

**用户反馈**："自动翻译开启后显示翻译失败"

1. **问题分析**：现象为自动翻译功能显示"翻译失败"；可能原因包括 API 请求失败、状态管理问题、消息传递问题
2. **创建子 Thread**：按主 Thread 任务拆解格式下发"翻译功能修复"任务（负责自动翻译相关功能；验收标准：自动翻译正常工作、结果正确显示、不影响悬停翻译、错误信息清晰明确）
3. **子 Thread 内部规划**：按子 Thread 任务定义格式限定允许修改 content.js，禁止修改 background.js、popup.js、manifest.json
4. **子 Agent 执行修复**
5. **验收结果**：子 Thread 主 Agent 验收子 Agent；主 Thread 验收子 Thread 整体结果
6. **交付用户**

## 超时问询机制

超时问询分为两级，检查时间均同步到用户界面并记录结果：

- **主 Thread → 子 Thread**：定期检查子 Thread 工作状态；超时时主动问询整体进度、阻塞问题和是否需要支持
- **子 Thread 主 Agent → 子 Agent**：定期检查子 Agent 状态；超过预估时间未收到通知时主动问询当前进度、遇到的困难或阻塞、是否需要额外支持、预计剩余时间

时间预估要求：子 Thread 主 Agent 分配子 Agent 任务时必须提供完成时间预估：

```
任务：<描述>
预计完成时间：<时间>
检查时间点：<时间>
超时阈值：<时间>
```

问询格式（两级通用）：

```
问询时间：<当前时间>
层级：<主 Thread / 子 Thread>
角色：<稳定职能名称>
任务：<任务描述>
预估完成时间：<预估时间>
当前状态：<状态描述>
问询原因：<超时/主动检查>
```

用户界面同步：显示子 Agent 工作状态和超时预警信息。
