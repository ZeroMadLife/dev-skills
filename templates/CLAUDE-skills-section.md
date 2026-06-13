# CLAUDE.md Skills 段落模板

将此内容插入到项目 `CLAUDE.md` 的合适位置（建议在"项目概述"之后）。

---

## Project Skills

本项目使用精选的开发 skills 补充标准工作流，存放在 `.shared-skills/` 目录（Git 子模块）。

### 已安装 Skills

#### logic-lens
**用途**：深度代码审查，检测竞态条件、空指针、安全漏洞、算法错误等传统 linter 无法发现的问题。

**何时使用**：
- 重构前审查复杂逻辑
- 合并 PR 前全面检查
- 面试前优化代码示例
- 涉及认证/授权/SQL 等安全敏感代码

**触发方式**：
- 自动：中度/重度任务时自动触发
- 手动：在需要时调用 `@logic-lens`

**示例**：
```
@logic-lens review InterfaceOutConfigServiceImpl.java focusing on OAuth2 retry logic
```

---

#### user-thoughts
**用途**：持久化项目架构决策、技术选型理由、已知权衡到 `.ustht/mdbase/`，确保跨会话上下文一致性。

**何时使用**：
- 做出架构决策后（如"OAuth2 只重试 1 次"）
- 技术选型讨论后（如"为什么选 MyBatis-Plus"）
- 发现权衡取舍后（如"竞态风险暂时接受"）
- 拒绝某方案后（如"考虑过 Caffeine 但放弃"）

**触发方式**：
- 自动：重度任务完成后自动记录
- 手动：明确说"记录这个决策"或调用 `/ustht`

**示例**：
```
/ustht 记录：OAuth2 只重试 1 次，因为 token 过期是瞬时故障，权限不足是持久故障
```

---

#### prompt-intensity（可选）
**用途**：自动判断任务复杂度（轻/中/重），决定触发哪些 skills 和读取哪些记忆维度。

**何时使用**：会话启动协议自动调用，无需手动触发。

**工作原理**：
- 轻度任务 → 不读记忆，不触发 skill
- 中度任务 → 读 rules + plans，触发 logic-lens
- 重度任务 → 读全部维度，触发 brainstorming + user-thoughts + logic-lens

---

### Skills 同步说明

`.shared-skills/` 是 Git 子模块，指向 `https://github.com/ZeroMadLife/dev-skills`。

**初始化（首次克隆项目）**：
```bash
git submodule update --init --recursive
```

**更新 skills（获取最新版本）**：
```bash
cd .shared-skills && git pull origin main
cd .. && git add .shared-skills && git commit -m "chore: update dev-skills"
```

**Claude Code 加载方式**：
- `.claude/skills/` 软链接到 `.shared-skills/`
- Claude Code 自动扫描并加载

---

### 架构决策记忆（.ustht/）

项目级架构决策和技术选型理由持久化在 `.ustht/mdbase/` 目录，确保跨会话上下文一致性。

**五个维度**：
- `rules.md` - 架构规则（如"OAuth2 只重试 1 次"）
- `dev-stack.md` - 技术栈选型理由（如"为什么选 MyBatis-Plus"）
- `tradeoffs.md` - 已知权衡（如"竞态风险暂时接受"）
- `rejected.md` - 被拒方案（如"考虑过 Caffeine 但放弃"）
- `plans.md` - 开发进度跟踪

**读取时机**：见"会话启动协议"。

**初始化**：
```bash
# user-thoughts skill 会自动创建 .ustht/ 目录
/ustht init
```

---

### 相关文档

- Skills 评估标准：`.shared-skills/docs/evaluation.md`
- 工具兼容性：`.shared-skills/docs/tool-mapping.md`
- 会话启动协议：见本文档顶部
