# AGENTS.md Skills 段落模板

将此内容插入到项目 `AGENTS.md` 的合适位置（建议在"飞书规范"段落之后）。

---

## Project Skills（项目级技能）

本项目使用精选的开发 skills 补充标准工作流，存放在 `.shared-skills/` 目录（Git 子模块）。

### 已安装 Skills

#### logic-lens
**用途**：深度代码审查，检测竞态条件、空指针、安全漏洞、算法错误等传统 linter 无法发现的问题。

**何时使用**：
- 重构前审查复杂逻辑
- 合并 PR 前全面检查
- 涉及认证/授权/SQL 等安全敏感代码

**触发方式（Codex 环境）**：
- Codex **不会**自动扫描 `.shared-skills/`
- 需要在任务开始时**显式读取** `.shared-skills/logic-lens/SKILL.md`
- 按 SKILL.md 的流程执行代码审查

**示例**：
```
先读取 .shared-skills/logic-lens/SKILL.md，然后按它的 9 大风险类别审查 InterfaceOutConfigServiceImpl.java 的 OAuth2 重试逻辑
```

---

#### user-thoughts
**用途**：持久化项目架构决策、技术选型理由、已知权衡到 `.ustht/mdbase/`，确保跨会话上下文一致性。

**何时使用**：
- 做出架构决策后（如"OAuth2 只重试 1 次"）
- 技术选型讨论后（如"为什么选 MyBatis-Plus"）
- 发现权衡取舍后（如"竞态风险暂时接受"）
- 拒绝某方案后（如"考虑过 Caffeine 但放弃"）

**触发方式（Codex 环境）**：
- Codex **不会**自动扫描 `.shared-skills/`
- 需要在任务开始时**显式读取** `.shared-skills/user-thoughts/SKILL.md`
- 按 SKILL.md 的工作流记录决策

**示例**：
```
先读取 .shared-skills/user-thoughts/SKILL.md，然后记录这个决策：OAuth2 只重试 1 次，因为 token 过期是瞬时故障，权限不足是持久故障
```

---

#### prompt-intensity（可选）
**用途**：自动判断任务复杂度（轻/中/重），决定触发哪些 skills 和读取哪些记忆维度。

**触发方式（Codex 环境）**：
- 读取 `.shared-skills/prompt-intensity/SKILL.md`
- 按其规则判断当前任务强度
- 根据判断结果决定后续动作

---

### Skills 加载说明（重要）

**Codex 当前行为**：
- ❌ Codex **不会自动扫描**项目级 `.shared-skills/` 或 `.claude/skills/`
- ✅ Codex 能读懂 `SKILL.md` 格式
- ✅ 需要在 `AGENTS.md` 中**显式引用**才会加载

**正确的使用方式**：
```
涉及代码审查时，先读取 .shared-skills/logic-lens/SKILL.md
涉及架构决策时，先读取 .shared-skills/user-thoughts/SKILL.md
```

**不要假设 Codex 会自动加载**，必须显式读取。

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

**Codex 读取方式**：
```
先读取 .ustht/mdbase/rules.md 和 .ustht/mdbase/plans.md，了解既定规则和当前进度
```

---

### 与飞书规范的关系

**优先级**：
1. 飞书知识库规范（强制，公司层面）
2. `.ustht/mdbase/` 记忆（项目层面，架构决策）
3. 本地代码实现（最具体）

**冲突处理**：
- 飞书规范 > 项目记忆 > 代码实现
- 如果发现冲突，以飞书规范为准，并更新项目记忆

---

### 相关文档

- Skills 评估标准：`.shared-skills/docs/evaluation.md`
- 工具兼容性：`.shared-skills/docs/tool-mapping.md`
- 会话启动协议：见本文档顶部

---

### 示例：完整的 Codex 工作流

**场景**：重构 OAuth2 重试逻辑

**步骤**：
```
1. 读取飞书文档（按 AGENTS.md 顶部的飞书规范流程）
2. 读取 .ustht/mdbase/rules.md（了解既定架构规则）
3. 读取 .ustht/mdbase/plans.md（了解当前进度）
4. 读取 .shared-skills/logic-lens/SKILL.md
5. 按 logic-lens 的 9 大风险类别审查现有代码
6. 提出修改方案
7. 读取 .shared-skills/user-thoughts/SKILL.md
8. 按 user-thoughts 工作流记录重构决策到 .ustht/mdbase/rules.md
```
