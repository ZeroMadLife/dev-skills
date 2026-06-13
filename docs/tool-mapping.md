# 工具名映射表

本文档定义如何编写跨 AI 编码助手（Claude Code / Codex / Cursor / Gemini）的通用 skills。

---

## 核心原则

**Skills 应该描述"做什么"，而不是"用什么工具"。**

❌ **错误示例**（绑定 Claude 专属工具）：
```markdown
Use the Read tool to open the file, then use Edit tool to modify it.
```

✅ **正确示例**（通用描述）：
```markdown
Read the target file, then apply changes using your agent's editing capability.
```

---

## 常见操作的通用描述

### 文件操作

| 操作 | Claude Code | Codex | Cursor | 通用描述 |
|------|------------|-------|--------|---------|
| **读取文件** | `Read` | file read | file view | "Read the file using your agent's file reading tool" |
| **写入新文件** | `Write` | write_file | create file | "Write a new file using your agent's file creation tool" |
| **编辑现有文件** | `Edit` | apply_patch | file edit | "Apply changes using your agent's file editing tool" |
| **列出目录** | `Bash("ls")` | exec_command | terminal | "List directory contents using your shell tool" |

### 搜索操作

| 操作 | Claude Code | Codex | Cursor | 通用描述 |
|------|------------|-------|--------|---------|
| **搜索文件** | `Bash("find")` | exec_command | search files | "Search for files using your agent's search tool or `find` command" |
| **搜索内容** | `Bash("rg")` | ripgrep / exec_command | grep | "Search file contents using ripgrep (`rg`) or grep" |
| **查找符号** | `Bash("rg")` | symbol search | go to definition | "Search for symbol definitions using your agent's code navigation tool" |

### Shell 操作

| 操作 | Claude Code | Codex | Cursor | 通用描述 |
|------|------------|-------|--------|---------|
| **运行命令** | `Bash` | exec_command | terminal | "Execute shell commands using your agent's shell tool" |
| **运行脚本** | `Bash("python script.py")` | exec_command | terminal | "Run the script using your shell tool: `python script.py`" |
| **Git 操作** | `Bash("git status")` | exec_command | terminal | "Use git commands via your shell tool: `git status`" |

### 任务管理

| 操作 | Claude Code | Codex | Cursor | 通用描述 |
|------|------------|-------|--------|---------|
| **创建任务** | `TaskCreate` | update_plan | N/A | "Track this as a task using your agent's task management tool (if available)" |
| **更新任务** | `TaskUpdate` | update_plan | N/A | "Update task status using your agent's task tool (if available)" |
| **列出任务** | `TaskList` | show_plan | N/A | "List current tasks using your agent's task tool (if available)" |

### 协作操作

| 操作 | Claude Code | Codex | Cursor | 通用描述 |
|------|------------|-------|--------|---------|
| **启动子 Agent** | `Agent` | N/A | N/A | "Delegate to a sub-agent if your environment supports parallel agents" |
| **多线程执行** | `Agent` (parallel) | N/A | N/A | "If your agent supports parallel execution, run these tasks concurrently" |

---

## 在 SKILL.md 中如何写 Tool Notes

每个跨平台 skill 应该有 **Tool Notes** 段落，说明如何适配不同工具。

### 模板

```markdown
## Tool Notes

This skill is designed to work across multiple AI coding assistants. Tool requirements:

**File operations:**
- Read files: Use your agent's file reading tool (Claude: `Read` / Codex: file read / Cursor: file view)
- Edit files: Use your agent's editing tool (Claude: `Edit` / Codex: apply_patch / Cursor: file edit)

**Search operations:**
- Search code: Use ripgrep (`rg`) or your agent's built-in search
- Navigate symbols: Use your agent's code navigation tool

**Shell operations:**
- Run commands: Use your agent's shell tool (Claude: `Bash` / Codex: exec_command / Cursor: terminal)

**Task management:**
- Track tasks: Use your agent's task tool if available (Claude: `TaskCreate` / Codex: update_plan)
- If task tools are unavailable, maintain a manual checklist in a TODO.md file

**No tool-specific commands:**
- Do NOT write commands like "Run the Read tool on file.py"
- Instead write "Read file.py using your agent's file reading capability"
```

### 实际示例：logic-lens

```markdown
## Tool Notes

This skill is designed to work across multiple AI coding assistants (Claude Code, Codex, Cursor, Gemini). It does NOT rely on tool-specific commands. Instead, it uses the agent's native capabilities:

- **File reading**: Use your agent's file reading tool (Claude: `Read` / Codex: file read / Cursor: file view)
- **Code navigation**: Use your agent's search/grep tool (Claude: `Bash` with `rg` / Codex: ripgrep / Cursor: search)
- **Output**: Present findings as structured text (severity levels + actionable suggestions)

This skill does NOT invoke editing tools directly. It only provides analysis and recommendations. The agent should present findings to the user, who decides whether to apply fixes.
```

---

## 如何测试跨平台兼容性

### 自检清单

在 skill 中搜索以下关键词，确保没有绑定专属工具：

❌ 需要替换的专属工具名：
- `Read tool`
- `Write tool`
- `Edit tool`
- `Bash tool`
- `TaskCreate`
- `Agent tool`

✅ 应该使用的通用表述：
- "your agent's file reading tool"
- "your agent's editing capability"
- "your shell tool"
- "your agent's task management tool (if available)"
- "delegate to a sub-agent if supported"

### 人工测试

在不同环境测试 skill 是否能正常工作：

1. **Claude Code 测试**：
   ```bash
   cd ~/project
   # 调用 skill，观察是否正常执行
   ```

2. **Codex 测试**：
   - 在 Codex 环境中显式引用 skill
   - 检查 Codex 是否能理解并执行

3. **跨会话测试**：
   - 新会话中调用 skill
   - 检查是否需要重新解释

---

## 附录：常见错误和修复

### 错误 1：硬编码工具名

❌ **错误**:
```markdown
1. Use the Read tool to open InterfaceOutConfigServiceImpl.java
2. Use the Edit tool to fix the bug
```

✅ **修复**:
```markdown
1. Read InterfaceOutConfigServiceImpl.java
2. Apply the fix using your agent's editing tool
```

---

### 错误 2：假设特定工具存在

❌ **错误**:
```markdown
Create a task with TaskCreate to track this work.
```

✅ **修复**:
```markdown
Track this work as a task using your agent's task management tool (if available). If task tools are not available, maintain a manual TODO.md file.
```

---

### 错误 3：写成命令而非描述

❌ **错误**:
```markdown
Run: Read("file.py")
```

✅ **修复**:
```markdown
Read file.py to understand its structure.
```

---

## 总结

编写跨平台 skill 的黄金法则：

1. **描述意图，不绑定工具**：说"读取文件"，不说"用 Read 工具"
2. **提供回退方案**：如果某工具不可用，提供手动替代方案
3. **加 Tool Notes 段落**：显式说明如何适配不同平台
4. **自检清单**：搜索专属工具名，全部替换成通用描述
