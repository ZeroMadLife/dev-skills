# dev-skills

**轻量级的跨 AI 编码助手开发 skills 架构**，解决跨会话上下文丢失和开发过程规则遗忘问题。

适用于：Claude Code / Codex / Cursor / Gemini

---

## 💡 它是干什么的？

解决两个核心问题：

1. **新会话上下文丢失** 😫  
   → 每次新会话都要重新解释项目背景、架构决策、技术选型

2. **开发过程规则遗忘** 😫  
   → 第 1 天定的规则（如"OAuth2 只重试 1 次"），第 30 天忘了为什么

**dev-skills 如何解决**：
- ✅ 项目记忆持久化（`.ustht/mdbase/`）：架构规则、技术选型、权衡取舍
- ✅ 会话启动协议：新会话自动读取记忆，恢复上下文
- ✅ 条件触发：轻度任务不读记忆（快），中度/重度任务读记忆（安全）

---

## 🎯 实测效果（2026-06-13 测评）

在数据中台项目的 24 条代表性 case 测试中：

| 指标 | 结果 |
|------|------|
| **分类准确率** | 100% |
| **高风险漏触发率** | 0% |
| **流程成本** | 5.50（比 superpowers 低 52.7%） |

**典型成功案例**：
- ✅ "顺手改一下动态 SQL" → 识别为 SQL 注入风险（Medium）
- ✅ "OAuth2 小改重试逻辑" → 触发 logic-lens，检查竞态条件
- ✅ 新会话自动读取 `rules.md`，遵守既定规则

详见：[测评报告](docs/skill-evaluation/2026-06-13-dev-skills-vs-superpowers.md)

---

## 🚀 5 分钟快速开始

### 1. 添加到你的项目

```bash
cd your-project
git submodule add https://github.com/ZeroMadLife/dev-skills .shared-skills
```

### 2. 集成到 AI 编码助手

**Claude Code**：
```bash
ln -s ../.shared-skills .claude/skills
# Claude Code 会自动扫描 .claude/skills/
```

**Codex / Cursor**：
在 `AGENTS.md` 顶部添加（Codex 不会自动扫描，需要显式引用）：
```markdown
## Project Skills

涉及代码审查时，先读取 `.shared-skills/logic-lens/SKILL.md`
涉及架构决策时，先读取 `.shared-skills/user-thoughts/SKILL.md`
```

### 3. 添加会话启动协议

复制 [`templates/session-bootstrap.md`](templates/session-bootstrap.md) 到你的 `CLAUDE.md` / `AGENTS.md` 顶部。

**核心机制**（条件读取）：
- **轻度任务**（查日志、小修改）→ 不读记忆，直接处理
- **中度任务**（重构、新功能）→ 读 rules + plans，触发 logic-lens
- **重度任务**（架构设计、多模块）→ 读全部维度，触发完整流程

### 4. 初始化项目记忆

```bash
mkdir -p .ustht/mdbase
touch .ustht/mdbase/{rules,dev-stack,tradeoffs,rejected,plans}.md
```

**五个维度**：
- `rules.md` - 架构规则（如"OAuth2 只重试 1 次"）
- `dev-stack.md` - 技术选型理由（如"为什么选 MyBatis-Plus"）
- `tradeoffs.md` - 已知权衡（如"竞态风险暂时接受"）
- `rejected.md` - 被拒方案（如"考虑过 Caffeine 但放弃"）
- `plans.md` - 开发进度跟踪

✅ **完成！** 新会话会根据任务复杂度自动读取记忆。

---

## 📦 包含的 Skills

### logic-lens
深度代码审查，检测竞态条件、空指针、安全漏洞等 9 大风险类别。

**何时触发**：中度/重度任务（重构、新功能、安全敏感代码）

### user-thoughts
持久化架构决策、技术选型理由、已知权衡到 `.ustht/mdbase/`。

**何时触发**：重度任务完成后，记录重要决策

### prompt-intensity
自动判断任务复杂度（轻/中/重），决定触发哪些 skills 和读取哪些记忆。

**何时触发**：会话启动协议自动调用

---

## 🆚 对比 superpowers

| 维度 | dev-skills | superpowers |
|------|-----------|-------------|
| **轻量级** | ✅ Markdown 文件 | ❌ MCP 服务器常驻 |
| **可控性** | ✅ 完全透明可编辑 | ❌ 黑盒动态注入 |
| **项目记忆** | ✅ `.ustht/` 项目特定 | ❌ 无项目记忆 |
| **跨工具** | ✅ Claude + Codex 都支持 | ⚠️ 主要为 Claude 设计 |
| **流程成本** | ✅ 5.50（轻量） | ❌ 11.62（偏重） |
| **适用场景** | **日常开发默认** | 重型工程任务 |

**建议**：dev-skills 作为日常默认，superpowers 保留用于 TDD、复杂重构等重型任务。

---

## 📖 使用建议

| 任务类型 | 推荐流程 |
|---------|---------|
| 查日志、读文件、解释代码 | 直接处理，不触发 skills |
| 单模块 bugfix、接口开发、SQL 改造 | dev-skills（rules + plans + logic-lens） |
| 架构设计、新子系统、技术选型 | dev-skills heavy flow |
| 明确要求 TDD、复杂重构、PR 前强审查 | dev-skills + superpowers |

---

## 📚 文档

- [完整接入指南](docs/SETUP.md)（详细步骤）
- [Skills 评估标准](docs/evaluation.md)（A/B/C/D 四维度）
- [工具兼容性](docs/tool-mapping.md)（Claude/Codex 工具名映射）
- [测评报告](docs/skill-evaluation/2026-06-13-dev-skills-vs-superpowers.md)（实测数据）

---

## 🔄 更新 Skills

```bash
cd .shared-skills && git pull origin main
cd .. && git add .shared-skills && git commit -m "chore: update dev-skills"
```

---

## 🤝 贡献

欢迎提交 PR 添加新 skills 或优化现有 skills。

**要求**：
1. 必须有 `Tool Notes` 段落说明跨平台兼容性
2. 不绑定特定工具名（遵循 [`docs/tool-mapping.md`](docs/tool-mapping.md)）
3. 提供清晰的使用示例
4. 说明何时使用、何时不使用

---

## 📄 许可证

MIT License

---

## 🔗 相关链接

- [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills) - 社区 skills 集合（本项目的 logic-lens 和 user-thoughts 来源于此）
- [Claude Code 官方文档](https://docs.anthropic.com/claude-code)
