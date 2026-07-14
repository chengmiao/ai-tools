---
name: advisor
description: Advisor 顾问模式 - 当前对话作为 Executor 负责实际工作，遇到连续失败 3 次或架构决策问题时，通过 Agent 工具调用更强的 Advisor 模型获取 400-700 token 战略建议。触发词：顾问模式、advisor、架构决策、方案选型、卡住了、连续失败
---

# Advisor 顾问模式

## 模型配置（在此修改）

| 角色 | 模型 | 如何修改 |
|------|------|----------|
| **Executor** | 当前对话所选模型（建议 sonnet） | 用 `/model` 命令切换对话模型 |
| **Advisor** | `claude-fable-5` | 修改 `.claude/agents/advisor.md` 中 frontmatter 的 `model` 字段 |

Advisor 的模型定义在自定义子 Agent 文件 `.claude/agents/advisor.md` 中，调用时通过 `subagent_type="advisor"` 使用，**无需手动切换对话模型**。

**回退规则**：如果 `subagent_type="advisor"` 不可用或模型名被拒绝，改用 `Agent(model="opus", ...)`（当前最强别名）。

---

## 角色定义

**你（当前对话）= Executor**，负责所有实际工作：写代码、读文件、跑命令、改 bug。

**Advisor = 上方配置的 ADVISOR_MODEL（默认 `claude-fable-5`）**，只在关键节点出场，通过 Agent 工具调用，输出 400-700 token 的战略建议后退出。

---

## 触发条件（满足任一即触发）

1. **连续失败 ≥ 3 次** — 同一问题用不同方法尝试了 3 次仍失败
2. **架构决策关键词** — 对话中出现以下词语时主动提议咨询：
   - 架构、重构、设计模式、方案选型、trade-off
   - 如何设计、哪种方案更好、最佳实践
   - 数据库设计、API 设计、模块拆分、性能瓶颈

---

## 触发时的操作

使用 Agent 工具调用 Advisor 子 Agent（模型已在 `.claude/agents/advisor.md` 中固定为 fable-5）：

```
Agent(
    subagent_type="advisor",
    description="Advisor 战略咨询",
    prompt="""你是战略顾问。Executor 遇到了问题需要你的指导。

## 当前任务
{当前任务的简要描述}

## 已尝试的方案
{列出已经尝试过的方法和失败原因}

## 需要建议的问题
{具体问题}

请给出 400-700 token 的战略建议：
- 诊断根本原因
- 推荐最佳路径（具体到使用什么方案/API/模式）
- 不需要写实现代码，Executor 负责执行
"""
)
```

---

## 收到建议后

1. 将 Advisor 的建议作为上下文继续工作
2. 不需要再次调用 Advisor（同一问题只咨询一次）
3. 如果建议解决了问题，继续执行；如果仍有疑问，可再次触发

---

## 成本说明

- 90%+ 的 token 消耗在 Executor（Sonnet 价格）
- Advisor 每次出场约 700 token（Fable 5 价格）
- 每个任务通常调用 1-2 次 Advisor
- 效果约等于 92% 的 Fable 5 能力，花费约 63% 的成本
