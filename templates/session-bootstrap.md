# 会话启动协议模板

将此内容复制到项目的 `CLAUDE.md` 和 `AGENTS.md` 的**最顶部**（在项目概述之前）。

---

## 会话启动协议（Session Bootstrap Protocol）

> **目的**：保证每次新会话能快速恢复项目上下文，并遵守既定架构规则。

### 触发规则（Conditional Loading）

在处理用户请求前，先判断任务复杂度：

#### 1️⃣ 轻度任务（Light Intensity）
**场景**：查询、看日志、读文件、小修改（typo/注释）、非开发任务

**动作**：
- ❌ 不读取 `.ustht/` 记忆
- ❌ 不触发任何 skill
- ✅ 直接处理请求

**示例**：
- "这个函数是干什么的？"
- "看看最近的 git log"
- "README 里有个错别字"

---

#### 2️⃣ 中度任务（Medium Intensity）
**场景**：重构、新功能、bug 修复、代码审查

**动作**：
- ✅ 读取 `.ustht/mdbase/rules.md`（既定架构规则，不可违反）
- ✅ 读取 `.ustht/mdbase/plans.md`（当前开发进度）
- ✅ 处理前触发 `logic-lens` 审查关键逻辑
- ✅ 完成后如有重要决策，调用 `user-thoughts` 记录

**示例**：
- "重构 OAuth2 重试逻辑"
- "给接入侧加运行日志查询接口"
- "修复这个 NPE bug"
- "合并前帮我 review 代码"

---

#### 3️⃣ 重度任务（Heavy Intensity）
**场景**：架构设计、多模块改动、技术选型、新子系统

**动作**：
- ✅ 读取 `.ustht/mdbase/` **全部维度**：
  - `rules.md` - 既定规则
  - `dev-stack.md` - 技术栈选型理由
  - `tradeoffs.md` - 已知权衡
  - `rejected.md` - 被拒方案（避免重复讨论）
  - `plans.md` - 开发进度
- ✅ 先触发 `brainstorming` 探索需求
- ✅ 讨论后触发 `user-thoughts` 记录决策
- ✅ 实现前触发 `logic-lens` 审查

**示例**：
- "设计认证授权架构"
- "要不要引入分布式锁？"
- "加一个新的数据接出模块"
- "OAuth2 重试有竞态，怎么解决？"

---

### 自动判断（可选）

如果项目安装了 `prompt-intensity` skill，可以自动判断任务强度：

```markdown
1. 调用 prompt-intensity 分类请求
2. 根据分类结果执行对应动作
3. 继续处理任务
```

如果未安装，手动按上述规则判断即可。

---

### 记忆维度说明

`.ustht/mdbase/` 的五个维度：

| 文件 | 记录内容 | 何时读取 |
|------|---------|---------|
| **rules.md** | 架构规则和约束 | 中度、重度 |
| **dev-stack.md** | 技术栈选型理由 | 重度 |
| **tradeoffs.md** | 已知权衡和妥协 | 重度 |
| **rejected.md** | 被拒绝的方案 | 重度 |
| **plans.md** | 开发进度跟踪 | 中度、重度 |

---

### 注意事项

1. **既定规则不可违反**：读取 `rules.md` 后，处理任务时必须遵守，不得擅自改变
2. **记录重要决策**：完成重度任务后，必须调用 `user-thoughts` 记录决策和原因
3. **简单任务不过度**：轻度任务不要浪费时间读记忆，直接处理
4. **新会话恢复上下文**：如果用户说"继续上次的工作"，按重度任务处理（读全部维度）

---

### 快速参考

```
轻度任务 → 不读记忆，直接处理
中度任务 → 读 rules + plans，用 logic-lens
重度任务 → 读全部维度，用 brainstorming + user-thoughts + logic-lens
```
