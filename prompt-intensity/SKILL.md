---
name: prompt-intensity
description: Use when starting a project session or when a development request may need memory loading, project skills, or heavier planning.
category: workflow
risk: safe
source: community
source_repo: ZeroMadLife/dev-skills
source_type: community
license: MIT
date_added: "2026-06-13"
author: ZeroMadLife
tags: [workflow, automation, context-management, session-bootstrap]
tools: [claude, codex, cursor, gemini]
---

# Prompt Intensity Judgment

Automatically classify incoming prompts by complexity to optimize skill triggering and memory loading.

## When to Use

Use this skill at the start of a project session, or when a request is ambiguous enough that the agent needs to decide whether to load project memory, trigger review skills, or enter a heavier planning flow. It determines:
- Which skills should be triggered
- Which memory dimensions (.ustht/mdbase/) should be loaded
- Whether to enter brainstorming mode for complex tasks

Do not invoke this skill for every small follow-up turn inside an already classified task. If the user only asks for a status update, a simple file read, a tiny wording change, or a direct command result, handle it directly.

## Classification Rules

### Light Intensity

**Characteristics:**
- Simple queries (查询、看日志、读文件)
- Small modifications (typo fixes, comment updates, single-line changes)
- Non-development tasks (search, explain, translate)

**Actions:**
- Do NOT load any .ustht/ memory
- Do NOT trigger any development skills
- Process directly

**Examples:**
- "这个函数是干什么的？"
- "看看最近的 git log"
- "README 里有个错别字，改成..."

---

### Medium Intensity

**Characteristics:**
- Refactoring single module/file
- Adding new feature to existing module
- Bug fixes requiring code analysis
- Code review before PR

**Actions:**
- Load `.ustht/mdbase/rules.md` + `.ustht/mdbase/plans.md`
- Trigger `logic-lens` before making changes
- Follow existing architectural rules

**Examples:**
- "重构 InterfaceOutConfigServiceImpl 的 OAuth2 重试逻辑"
- "给接入侧加一个运行日志查询接口"
- "修复这个 NPE bug"
- "合并前帮我 review 一下代码"

---

### Heavy Intensity

**Characteristics:**
- Architectural design or redesign
- Multi-module changes
- Technology stack selection
- New subsystem implementation
- Cross-cutting concerns (security, performance, observability)

**Actions:**
- Load ALL `.ustht/mdbase/` dimensions (rules + dev-stack + tradeoffs + rejected + plans)
- Trigger `brainstorming` to explore requirements
- Trigger `user-thoughts` to record decisions after discussion
- Trigger `logic-lens` for final review

**Examples:**
- "设计一个接口治理平台的认证授权架构"
- "我们要不要引入分布式锁？用 Redisson 还是 Zookeeper？"
- "这个项目要加一个新的数据接出模块"
- "OAuth2 重试有竞态条件，怎么解决？"

---

## Implementation

### Decision Logic

```
1. Parse user prompt
2. Check for keywords:
   - Light: "查询", "看", "读", "是什么", "解释", "翻译", typo, comment
   - Heavy: "设计", "架构", "选型", "要不要", "怎么办", multi-module, new subsystem
   - Medium: everything else in development context

3. If unclear, default to Medium (safer)

4. Return classification + actions
```

### Output Format

```markdown
**Prompt Intensity: [LIGHT/MEDIUM/HEAVY]**

Triggered actions:
- Load memory: [none / rules+plans / all dimensions]
- Trigger skills: [none / logic-lens / brainstorming+user-thoughts+logic-lens]
- Rationale: [one-sentence explanation]
```

---

## Tool Notes

This skill does NOT require any specific tools. It only performs text analysis and returns a classification result. The actual memory loading and skill triggering are handled by the session bootstrap protocol in CLAUDE.md/AGENTS.md.

---

## Integration with Session Bootstrap Protocol

This skill works in conjunction with the session bootstrap protocol defined in project-level CLAUDE.md/AGENTS.md:

```markdown
## Session Bootstrap Protocol (Conditional Trigger)

At the start of a new project session, or when the current request changes task intensity:
1. Invoke prompt-intensity skill to classify the request
2. Based on classification:
   - LIGHT → skip memory loading, process directly
   - MEDIUM → load .ustht/mdbase/rules.md + plans.md
   - HEAVY → load all .ustht/mdbase/ dimensions
3. Follow triggered skills as suggested
```

---

## Examples

### Example 1: Light

**User:** "git status 显示什么？"

**Classification:**
```
Prompt Intensity: LIGHT
Actions: none
Rationale: Simple query, no development work
```

---

### Example 2: Medium

**User:** "接出侧推送失败时的日志格式不对，改一下"

**Classification:**
```
Prompt Intensity: MEDIUM
Actions:
- Load: rules.md + plans.md
- Trigger: logic-lens (before changes)
Rationale: Single-module bug fix
```

---

### Example 3: Heavy

**User:** "我们要给数据中台加一个数据血缘追踪功能，你觉得怎么设计？"

**Classification:**
```
Prompt Intensity: HEAVY
Actions:
- Load: all .ustht/mdbase/ dimensions
- Trigger: brainstorming → user-thoughts → logic-lens
Rationale: New subsystem design requiring architectural decisions
```

---

## Limitations

- Classification is heuristic-based and may misclassify edge cases
- When in doubt, it defaults to MEDIUM (safer but may load unnecessary memory)
- Does not handle multi-part requests (e.g., "查日志 + 重构代码") - treats as highest intensity found

---

## Related Skills

- **brainstorming**: Triggered for HEAVY intensity tasks
- **user-thoughts**: Triggered for HEAVY intensity to record decisions
- **logic-lens**: Triggered for MEDIUM and HEAVY intensity
