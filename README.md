# dev-skills

精选的跨平台开发 skills，适用于 Claude Code / Codex / Cursor / Gemini。

## 理念

- **轻量级** > 重量级（文件 > MCP 服务器）
- **项目特定** > 通用（CLAUDE.md > 通用 skills）
- **可控** > 黑盒（Markdown > 动态注入）
- **补充** > 替代（填补空白，不替换现有流程）

## 已包含的 Skills

### logic-lens
深度代码审查，检测竞态条件、空指针、安全漏洞等 9 大风险类别。

**何时使用**：重构前、合并 PR 前、安全敏感代码审查

### user-thoughts
持久化架构决策、技术选型理由、已知权衡到 `.ustht/mdbase/`，确保跨会话上下文一致性。

**何时使用**：做出架构决策、技术选型、发现权衡取舍、拒绝方案后

### prompt-intensity
自动判断任务复杂度（轻/中/重），决定触发哪些 skills 和读取哪些记忆维度。

**何时使用**：会话启动协议自动调用

---

## 快速开始

### 作为 Git 子模块接入项目

```bash
cd your-project
git submodule add https://github.com/ZeroMadLife/dev-skills .shared-skills
```

### Claude Code 集成

```bash
# 创建软链接
ln -s ../.shared-skills .claude/skills

# Claude Code 会自动扫描 .claude/skills/
```

### Codex 集成

Codex **不会自动扫描**项目级 skills 目录，需要在 `AGENTS.md` 中显式引用：

```markdown
## Project Skills

涉及以下场景时，先读取对应 SKILL.md：

- 代码审查：`.shared-skills/logic-lens/SKILL.md`
- 架构决策：`.shared-skills/user-thoughts/SKILL.md`
```

---

## 会话启动协议

将 `templates/session-bootstrap.md` 的内容复制到项目的 `CLAUDE.md` 和 `AGENTS.md` 顶部。

**核心机制**：根据任务复杂度条件读取记忆

- **轻度任务**（查询、小修改）→ 不读记忆，直接处理
- **中度任务**（重构、新功能）→ 读 rules + plans，触发 logic-lens
- **重度任务**（架构设计、多模块）→ 读全部维度，触发 brainstorming + user-thoughts + logic-lens

---

## 目录结构

```
dev-skills/
├── README.md                      # 本文件
├── logic-lens/
│   └── SKILL.md                   # 代码审查 skill
├── user-thoughts/
│   ├── SKILL.md                   # 架构决策记录 skill
│   └── scripts/                   # Python 辅助脚本
├── prompt-intensity/
│   └── SKILL.md                   # 任务强度判断 skill
├── docs/
│   ├── evaluation.md              # Skills 有效性评估标准（A/B/C/D）
│   └── tool-mapping.md            # Claude/Codex 工具名映射表
└── templates/
    ├── session-bootstrap.md       # 会话启动协议模板
    ├── CLAUDE-skills-section.md   # CLAUDE.md 的 skills 段落模板
    └── AGENTS-skills-section.md   # AGENTS.md 的 skills 段落模板
```

---

## user-thoughts 五维度

`.ustht/mdbase/` 的五个维度：

- `rules.md` - 架构规则（如"OAuth2 只重试 1 次"）
- `dev-stack.md` - 技术栈选型理由（如"为什么选 MyBatis-Plus"）
- `tradeoffs.md` - 已知权衡（如"竞态风险暂时接受"）
- `rejected.md` - 被拒方案（如"考虑过 Caffeine 但放弃"）
- `plans.md` - 开发进度跟踪

---

## Skills 评估标准

采用 **A/B/C/D 四维度结合评估**：

- **A. 使用频率**：每周统计调用次数
- **B. 时间节省**：对比有/无 skill 的耗时
- **C. 输出质量**：主观评分（高/中/低价值）
- **D. Prompt 强度自动判断**：轻/中/重度决定触发哪些 skill

详见 `docs/evaluation.md`。

---

## 工具兼容性

所有 skills 遵循**通用描述原则**，不绑定特定工具名：

❌ 错误：`Use the Read tool to open the file`
✅ 正确：`Read the file using your agent's file reading tool`

详见 `docs/tool-mapping.md`。

---

## 更新 Skills

```bash
cd .shared-skills
git pull origin main
cd ..
git add .shared-skills
git commit -m "chore: update dev-skills"
```

---

## 完整接入指南

### 1. 添加 Git 子模块

```bash
cd ~/your-project
git submodule add https://github.com/ZeroMadLife/dev-skills .shared-skills
```

### 2. 集成到 Claude Code

```bash
ln -s ../.shared-skills .claude/skills
```

### 3. 更新 CLAUDE.md

复制 `templates/session-bootstrap.md` 到 `CLAUDE.md` 顶部

复制 `templates/CLAUDE-skills-section.md` 到 `CLAUDE.md` 合适位置

### 4. 更新 AGENTS.md（如果使用 Codex）

复制 `templates/session-bootstrap.md` 到 `AGENTS.md` 顶部

复制 `templates/AGENTS-skills-section.md` 到 `AGENTS.md` 合适位置

### 5. 初始化 .ustht/

```bash
# user-thoughts skill 会自动创建
# 或手动创建：
mkdir -p .ustht/mdbase
touch .ustht/mdbase/{rules,dev-stack,tradeoffs,rejected,plans}.md
```

### 6. 写入已有架构决策

把已有的架构决策写入 `.ustht/mdbase/` 对应维度。

---

## 贡献

欢迎提交 PR 添加新 skills 或优化现有 skills。

**添加新 skill 的要求**：
1. 必须有 `Tool Notes` 段落说明跨平台兼容性
2. 不绑定特定工具名（遵循 `docs/tool-mapping.md`）
3. 提供清晰的使用示例
4. 说明何时使用、何时不使用

---

## 许可证

MIT License

---

## 相关链接

- [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills) - 社区 skills 集合（本项目的 logic-lens 和 user-thoughts 来源于此）
- [Claude Code 官方文档](https://docs.anthropic.com/claude-code)
- [设计文档](https://github.com/ZeroMadLife/dev-skills/wiki) - 详细的架构设计说明
