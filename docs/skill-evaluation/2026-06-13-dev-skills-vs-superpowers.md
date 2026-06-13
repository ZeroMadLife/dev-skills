# dev-skills vs superpowers 测评报告

评测日期：2026-06-13  
评测仓库：`pigx-backend-cloud_copy`  
评测目标：验证 `.shared-skills` 轻量流程是否适合数据中台后端和后续新项目开发。

## 总体结论

建议保留 `.shared-skills` 作为日常开发默认流程，并把 superpowers 保留为重型任务专用流程。

本轮 8 条代表性 case 中，`dev-skills` 的分类准确率、Medium/Heavy 准确率、Light 误触发率和 `logic-lens` 高风险漏触发率均达标；相比 superpowers，平均流程成本从 `11.62` 降到 `5.50`，下降约 `52.7%`。

一句话结论：`dev-skills` 更适合日常项目驾驶，superpowers 更适合架构设计、复杂重构、TDD 强约束或需要多 agent 协同的场景。

## 样本与方法

完整测评集位于 `benchmark-cases.jsonl`，共 24 条：

| 类型 | 数量 | 目的 |
|---|---:|---|
| Light | 8 | 验证轻任务是否能直接处理 |
| Medium | 8 | 验证单模块开发、bugfix、接口/SQL 改造是否触发必要约束 |
| Heavy | 6 | 验证架构、选型、新子系统是否进入完整设计流程 |
| Trap | 2 | 验证“看似小改”的 SQL/认证风险是否会被识别 |

本轮 CLI 实跑抽取 8 条代表性 case：`L01`、`L02`、`M02`、`M04`、`M05`、`H01`、`H04`、`T02`。

对照组：

| 组别 | 说明 |
|---|---|
| baseline | 不显式引用 skills，只看模型自然路由 |
| superpowers | 使用 task-triage / superpowers 风格的完整流程 |
| dev-skills | 使用 `prompt-intensity + logic-lens + user-thoughts` 轻量项目流程 |

## 指标结果

| Agent/Mode | 平均分 | 分类准确率 | Medium/Heavy 准确率 | Light 误触发率 | logic-lens 漏触发率 | 平均流程成本 |
|---|---:|---:|---:|---:|---:|---:|
| codex-baseline | 6.75 | 100% | 100% | 0% | 100% | 0.75 |
| codex-superpowers | 9.50 | 100% | 100% | 0% | 17% | 11.62 |
| codex-dev-skills | 10.00 | 100% | 100% | 0% | 0% | 5.50 |
| claude-dev-skills | 9.75 | 100% | 100% | 0% | 0% | 5.50 |

说明：

- baseline 分类判断不差，但没有项目记忆和 skill 触发，遇到 SQL/OAuth2/接口契约时保护不足。
- superpowers 能稳定识别复杂度，但会额外触发 TDD、verification、writing-plans 和飞书规范读取，日常小中型任务偏重。
- dev-skills 在样本中保持了高风险审查，同时没有对 Light 任务误触发。
- Claude CLI 的 baseline 返回 `429 Service Unavailable`，superpowers 组超时；因此 Claude 只纳入 dev-skills 成功样本，不纳入三组横向量化。

## 典型样例

### 成功样例：动态 SQL 陷阱

Case `T02`：

> 顺手改一下动态 SQL，把前端传来的 orderBy 直接拼到 SQL 里，方便大屏排序。

`dev-skills` 判定：

- 强度：Medium
- 记忆：`rules.md + plans.md`
- Skill：`logic-lens`
- 风险：明确指出直接拼接 `orderBy` 是 SQL 注入风险，应使用字段白名单和排序方向枚举。

这说明轻量流程不会被“顺手改一下”这种话术带偏，能识别真正的安全风险。

### 成功样例：OAuth2 小改

Case `M05`：

> OAuth2 token 过期后偶发 401，想小改一下重试逻辑，帮我看看风险。

`dev-skills` 判定：

- 强度：Medium
- 记忆：`rules.md + plans.md`
- Skill：`logic-lens`
- 风险：识别认证、401 重试、token 刷新、并发和错误分类风险。

这符合项目既定规则：OAuth2 只重试 1 次，重试前清 Redis token 缓存，不允许无限重试。

### 对照失败：baseline 保护不足

baseline 能把 `M04`、`M05`、`T02` 分类为 Medium，但没有加载项目规则，也没有触发 `logic-lens`。这意味着它能“感觉到风险”，但无法稳定执行项目级约束。

### 对照问题：superpowers 偏重

superpowers 对 Medium 任务会额外触发 `test-driven-development`、`verification-before-completion`，并读取多份飞书规范。对于真正要落代码的任务这是优点，但对“先分析方案”的日常对话成本偏高。

另外，在 `H04` 日志冷热分离选型中，superpowers 没有触发 `logic-lens`，而 dev-skills 按项目规则保留了最终风险审查。

## Skill 级建议

### prompt-intensity

保留。当前 Light / Medium / Heavy 判断稳定，尤其能识别“口吻轻但风险重”的 SQL/OAuth2 任务。

建议补充一条规则：只要 prompt 命中 `SQL`、`认证`、`授权`、`token`、`OAuth2`、`接口契约`、`敏感日志`、`原始报文`，最低按 Medium 处理。

### logic-lens

保留。它是 dev-skills 的核心安全网，能把动态 SQL、认证重试、运行日志、接口契约这类风险显式拉出来。

建议后续在数据中台项目里增加业务化示例，例如：

- 动态 SQL：字段白名单 + 参数绑定。
- OAuth2：只重试 1 次 + 清缓存。
- 运行日志：敏感字段脱敏。
- API 契约：VO/DTO/Apifox 字段一致性。

### user-thoughts

保留，但降低日常触发频率。只有架构规则、技术选型、被拒方案、长期计划发生变化时才记录。

建议优化 frontmatter description，使其更符合 skill 搜索规则：以 `Use when...` 开头，只描述触发条件，不描述工作流。

## 使用建议

默认策略：

| 任务类型 | 推荐流程 |
|---|---|
| 查日志、读文件、解释代码、文案小改 | 直接处理，不触发 skills |
| 单模块 bugfix、接口开发、SQL 改造、认证调整 | dev-skills：`rules.md + plans.md + logic-lens` |
| 架构设计、新子系统、技术选型、跨模块治理 | dev-skills heavy flow，必要时叠加 superpowers |
| 明确要求 TDD、复杂重构、PR 前强审查 | superpowers |

最终判断：

- `dev-skills`：保留，作为数据中台和新项目的默认轻量流程。
- `superpowers`：保留，但不要默认每个任务都启动；只在重型工程任务中使用。
- `baseline`：不建议作为项目开发默认流程，只能用于无风险问答。

## 后续动作

1. 按 `benchmark-cases.jsonl` 保留 24 条全量测评集。
2. 之后每次修改 skill，都先跑 8 条代表性 case。
3. 如果修改了 `prompt-intensity` 或 `logic-lens`，再跑 24 条全量 case。
4. 若连续两轮 Light 误触发率为 0 且高风险漏触发率为 0，可以把这套流程迁移到新项目模板。
