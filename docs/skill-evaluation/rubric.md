# dev-skills vs superpowers 评分标准

本文档用于评估 `.shared-skills` 轻量流程是否能替代日常开发里的 superpowers 重流程。

## 评估对象

- `baseline`：不显式引用 skills，只观察模型自然路由。
- `superpowers`：使用 `using-superpowers` / `task-triage` 风格的完整流程。
- `dev-skills`：使用项目轻量流程：`prompt-intensity`、`logic-lens`、`user-thoughts`。

## 单条 Case 评分

每条 case 满分 10 分。

| 维度 | 分值 | 判定方式 |
|---|---:|---|
| 强度分类 | 2 | 输出的 light / medium / heavy 与期望一致得 2 分；相邻误判得 1 分；严重误判得 0 分 |
| 记忆加载 | 2 | 期望记忆文件完全匹配得 2 分；多读但不影响轻任务得 1 分；漏读关键记忆得 0 分 |
| Skill 触发 | 2 | 期望 skills 完全匹配得 2 分；多触发轻微过重得 1 分；漏触发高风险 skill 得 0 分 |
| 风险识别 | 2 | 正确识别 SQL、认证、API 契约、敏感日志等风险得 2 分；泛泛提醒得 1 分；漏掉得 0 分 |
| 成本控制 | 1 | 流程与任务规模匹配得 1 分；明显过度流程得 0 分 |
| 输出可执行性 | 1 | 结论清晰、下一步明确得 1 分；空泛得 0 分 |

## 汇总指标

| 指标 | 计算方式 | 通过线 |
|---|---|---:|
| Light 误触发率 | Light case 中触发 memory/skills/heavy flow 的比例 | <= 10% |
| Medium/Heavy 分类准确率 | Medium + Heavy case 中分类正确比例 | >= 85% |
| 高风险漏触发率 | auth/sql/api-contract/sensitive-data case 中未触发 `logic-lens` 的比例 | 0% |
| 平均流程成本 | 每条 case 声明的 memory + skill 数量加权 | dev-skills < superpowers |
| 平均单条得分 | 总分 / case 数 | >= 8 |

## 成本模型

为方便对比，使用轻量成本分：

| 动作 | 成本 |
|---|---:|
| 读取一个 `.ustht/mdbase` 文件 | 1 |
| 触发一个 skill | 2 |
| 进入 brainstorming / heavy planning | 3 |
| 不必要的全量 mdbase 加载 | 5 额外惩罚 |

成本分不是 token 精确值，只用于比较相同 case 下不同流程的相对负担。

## 决策标准

- **保留**：dev-skills 达到通过线，且平均成本低于 superpowers。
- **优化**：dev-skills 分类准确率达标，但 Light 误触发或高风险触发规则不稳定。
- **移除或降级**：dev-skills 高风险漏触发率不为 0，且通过补充规则仍无法稳定修复。

## 人工复核规则

- 对 CLI 输出只看最终路由决策，不评价模型写作风格。
- 如果模型没有显式说读取哪些文件，按未满足记忆加载计。
- 如果模型说“这是小改，不需要审查”，但 case 含 `auth`、`sql`、`api-contract`、`sensitive-data`，按高风险漏触发计。
- 如果模型拒绝危险需求并解释原因，即使没有给实现步骤，风险识别仍可满分。
