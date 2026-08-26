---
description: 启用并强制遵循 agent-dev-workflow 规范
---

# `/mwork`：启用 Agent 工作流

你必须把本命令视为当前 thread 的工作方式切换，而不是业务需求。严格按以下顺序执行，不得跳过或假装完成：

1. 定位并读取一份当前 thread 可访问的 `agent-dev-workflow` 规范副本。按以下优先级查找：
   - 当前项目根目录的 `.claude/skills/agent-dev-workflow/SKILL.md`；
   - 当前项目 `.claude/commands/` 的相邻规范路径（例如其上级 `.claude/skills/agent-dev-workflow/SKILL.md`）；
   - 当前用户级 Claude 配置目录中的 `skills/agent-dev-workflow/SKILL.md`（通常为 `~/.claude/skills/agent-dev-workflow/SKILL.md`）；
   - 本命令文件所在配置目录可访问的同名 `skills/agent-dev-workflow/SKILL.md`。
2. 读取该 `SKILL.md` 中引用的规范文档；若引用路径是相对路径，必须相对于找到的 `SKILL.md` 所在目录解析。至少读取 `references/development-workflow.md`、`references/version-management.md`、`references/thread-management.md`、`references/agent-roles.md`、`references/testing.md` 和 `references/delivery.md`（若这些文件存在）。
3. 如果找不到 `SKILL.md`、无法读取它，或规范所需的引用文件缺失导致无法可靠执行，立即停止，不执行用户请求的后续工作，并明确告诉用户：未找到/无法读取的路径、尚未启用规范，以及需要用户提供或安装规范副本。不得声称 `/mwork` 已生效。
4. 规范读取成功后，将其作为当前 thread 后续所有响应、分析、任务拆解、Agent 分工、代码修改、验收、测试和交付的强制规则。不得用本命令覆盖规范中的限制；遇到不确定事项按规范停止并汇报。若规范要求分配任务执行 Agent、质量验证 Agent 或维护工作流记录，必须照做。
5. 只有在完成上述读取和确认后，才处理本条 `/mwork` 命令中 `/mwork` 前缀之后的可选参数。该剩余文本只是附加上下文或待处理请求，不是启用条件，也不能改变上述优先级和强制规则；为空时仅确认规范已启用并等待下一条用户消息。

启用成功时，用中文简要确认：已读取的规范根路径、当前采用的工作流角色/状态，以及后续将严格遵循该规范。不要把 `/mwork` 前缀当作业务需求文本。除非用户明确指定其他语言，后续用户端说明默认使用中文。

## 本次命令的可选参数

`$ARGUMENTS`

## 安装到其他项目

要在其他项目中使用本命令，请将本文件放入目标项目的 `.claude/commands/mwork.md`，并确保该项目或用户级配置中存在 `agent-dev-workflow/SKILL.md` 及其 references。也可以将本文件安装为 `~/.claude/commands/mwork.md`，再将规范安装到 `~/.claude/skills/agent-dev-workflow/`，从而在多个项目中复用。
