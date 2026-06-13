# dev-skills 完整接入指南

本文档说明如何把 `dev-skills` 接入到一个项目中，并让 Claude Code、Codex、Cursor 等 AI 编码助手稳定读取项目级 skills 和项目记忆。

## 1. 添加 Git 子模块

在目标项目根目录执行：

```bash
git submodule add https://github.com/ZeroMadLife/dev-skills .shared-skills
git submodule update --init --recursive
```

推荐使用 `.shared-skills` 作为统一目录名，便于 Claude Code、Codex 和 Cursor 共享同一份 skill 内容。

## 2. Claude Code 接入

Claude Code 可以自动扫描 `.claude/skills/`，因此用软链接指向共享 skill 目录即可：

```bash
mkdir -p .claude
ln -s ../.shared-skills .claude/skills
```

如果系统不支持软链接，也可以复制 `.shared-skills` 到 `.claude/skills`，但后续需要手动同步更新。

## 3. Codex / Cursor 接入

Codex 不会自动扫描项目级 `.shared-skills/`，需要在 `AGENTS.md` 或项目规则文件中显式引用。

推荐放在 `AGENTS.md` 顶部：

```markdown
## Project Skills

本项目使用 `.shared-skills/` 中的项目级 skills。

- 涉及代码审查、复杂业务逻辑、认证、SQL、API 契约时，先读取 `.shared-skills/logic-lens/SKILL.md`。
- 涉及架构决策、技术选型、长期计划、被拒方案时，先读取 `.shared-skills/user-thoughts/SKILL.md`。
- 新会话或任务强度不明确时，按 `.shared-skills/prompt-intensity/SKILL.md` 判断 light / medium / heavy。
```

## 4. 添加会话启动协议

将 [`../templates/session-bootstrap.md`](../templates/session-bootstrap.md) 的内容复制到项目的 `CLAUDE.md` / `AGENTS.md` 顶部。

推荐策略：

| 任务强度 | 场景 | 动作 |
| --- | --- | --- |
| Light | 查日志、读文件、解释代码、小文案修改 | 不读记忆，不触发 skills |
| Medium | 单模块 bugfix、接口开发、SQL 改造、认证调整 | 读取 `rules.md + plans.md`，必要时触发 `logic-lens` |
| Heavy | 架构设计、新子系统、技术选型、跨模块治理 | 读取全部 mdbase 维度，使用 `brainstorming + user-thoughts + logic-lens` |

## 5. 初始化项目记忆

在目标项目中创建 `.ustht/mdbase/`：

```bash
mkdir -p .ustht/mdbase
touch .ustht/mdbase/{rules,dev-stack,tradeoffs,rejected,plans}.md
```

五个维度建议这样使用：

| 文件 | 内容 |
| --- | --- |
| `rules.md` | 已确定、后续不可随意违反的架构规则 |
| `dev-stack.md` | 技术栈选择和原因 |
| `tradeoffs.md` | 已接受的权衡和风险 |
| `rejected.md` | 讨论过但拒绝的方案 |
| `plans.md` | 当前开发进度和后续计划 |

## 6. 验证接入是否生效

可以用以下三类 prompt 做快速检查：

```text
看一下 git status，告诉我哪些文件有改动，不要做任何修改。
```

期望：Light，不读取 `.ustht/`，不触发 project skills。

```text
OAuth2 token 过期后偶发 401，想小改一下重试逻辑，帮我看看风险。
```

期望：Medium，读取 `rules.md + plans.md`，触发 `logic-lens`。

```text
我们要新增一个接出适配器插件机制，后续不同第三方可以按插件扩展，你帮我做架构设计。
```

期望：Heavy，读取全部 mdbase 维度，并进入设计/决策记录流程。

完整测评集见 [`skill-evaluation/benchmark-cases.jsonl`](skill-evaluation/benchmark-cases.jsonl)。

## 7. 更新 dev-skills

在接入项目中更新子模块：

```bash
cd .shared-skills
git pull origin main
cd ..
git add .shared-skills
git commit -m "chore: update dev-skills"
```

如果团队内有多个项目复用同一套 skills，建议先在一个项目中跑完测评集，再同步到其它项目。

