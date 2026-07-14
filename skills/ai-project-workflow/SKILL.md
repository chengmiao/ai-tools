---
name: ai-project-workflow
description: >-
  Use when: 初始化项目, 开始新功能开发, 拆分需求为feature, 制定实现计划,
  完成功能需要总结经验, 检查开发进度, 项目架构设计, 代码审查流程,
  调试连续失败需要升级, 需要子Agent协作分工, 复杂任务拆解执行.
when_to_use: >-
  Triggers on: 项目初始化, 新项目搭建, 开发流程, 需求分析, 功能拆分,
  计划制定, 经验总结, 项目管理, feature 管理, 开发规范,
  project init, project setup, development workflow, feature planning,
  plan generation, experience summary, code review workflow,
  debugging escalation, subagent dispatch, task breakdown.
---

# AI Agent 项目开发工作流

## 何时使用

- 初始化一个新项目，需要建立结构化的开发流程
- 开始一个复杂功能的开发，需要 Plan → Work → Review → Compound 循环
- 需要将原始需求拆分为可执行的 feature
- 完成一个功能后需要总结经验并归档
- 需要检查当前项目的开发状态和进度

## 项目目录结构

使用 `.ai-workflow/` 目录作为 Agent 开发的核心工作区：

```
.ai-workflow/
├── requirements/         # 原始需求（业务需求 + 技术需求）
│   └── README.md         # 需求编写规范和模板
├── context/              # 会话上下文（Agent 开发时默认加载）
│   ├── spec.md           # 核心开发规范（MVP First、代码质量、测试标准）
│   ├── arc.md            # 项目架构方案（技术选型、目录结构、组件说明）
│   └── constraint.md     # Agent 开发时的约束条件
├── features/             # 功能需求（由 Agent 从原始需求分析生成）
│   ├── active/           # 当前正在开发的功能（建议 2-5 个）
│   │   └── README.md     # Feature 文件格式模板
│   └── done/             # 已完成归档（检索与参考）
├── plan/                 # 实现方案
│   └── README.md         # Plan 编写规范和模板
├── exp/                  # 经验教训（按 feature 建立经验链）
│   ├── README.md         # 经验记录规范和模板
│   └── <feature-name>/   # 每个 feature 一个子目录，内含版本化经验链
│       ├── v1.md         # 初始经验
│       ├── v2.md         # 后续迭代经验（必须引用 v1）
│       └── ...           # 每次涉及该 feature 的开发都追加新版本
```

## Reference 文件

本 skill 的 `references/` 目录保存项目根目录 `.ai/` 下各工作区 README.md 的标准模板，作为初始化项目工作流目录的唯一模板来源：

| Reference | 初始化时生成到 |
|---|---|
| `references/requirements.md` | `.ai-workflow/requirements/README.md` |
| `references/context.md` | `.ai-workflow/context/README.md` |
| `references/features-active.md` | `.ai-workflow/features/active/README.md` |
| `references/plan.md` | `.ai-workflow/plan/README.md` |
| `references/exp.md` | `.ai-workflow/exp/README.md` |

修改工作流模板时，应同步更新对应的 reference 文件；初始化项目时直接使用 reference 文件内容生成目标 README.md，不要重新编写或创建另一套模板。

## 开发循环

```
        ┌─────────────────────────────────────┐
        │                                     │
        ▼                                     │
    ┌───────┐         ┌──────┐         ┌─────────┐
    │ Plan  │────────►│ Work │────────►│ Review  │
    └───────┘         └──────┘         └─────────┘
        ▲                                     │
        │                                     ▼
        │                              ┌─────────┐
        └──────────────────────────────│Compound │
                                       └─────────┘
```

### Phase 1: Plan

**输入**: `features/active/` 中的功能需求
**输出**: `plan/` 中的详细实现方案

操作步骤：
1. 读取 `context/` 下的 spec.md、arc.md、constraint.md 了解项目约束
2. 读取目标 feature 文件，理解需求细节
3. 读取 `exp/<feature-name>/` 中的经验链，从最新版本向前追溯，理解该 feature 的决策演变历史，避免重复踩坑
4. 生成分步实现计划，包含明确的验收标准
5. 将计划写入 `plan/` 目录

#### 计划粒度标准（"无判断力新人"原则）

计划要详细到一个**没有项目背景、没有品味、不愿测试的热情新人**也能执行：

- **每个步骤 2-5 分钟**："写失败测试"是一步，"运行测试确认失败"是另一步
- **禁止占位符**：每个步骤必须包含实际代码、实际命令、预期输出
- **明确文件路径**：指明每个步骤要修改的确切文件和代码位置
- **包含验证命令**：每个步骤结束后如何确认正确完成（运行什么命令、期望什么输出）
- **包含回退方案**：如果步骤失败，如何回退到上一个已知正确状态

示例对比：
```
❌ 差的计划步骤：  "实现用户认证模块"
✅ 好的计划步骤：
   步骤 3a: 在 src/auth/handler.rs 中添加 verify_token 函数
   - 代码：[实际代码]
   - 步骤 3b: 运行 `cargo test auth::tests::test_verify_token`
   - 预期输出：test result: ok. 1 passed
   - 失败回退：检查 JWT_SECRET 环境变量是否设置
```

<HARD-GATE>
计划未经用户明确确认之前，不得开始任何编码工作。
"用户说了大概要什么"不等于"用户确认了计划"。
你必须展示完整计划并获得明确批准后才能进入 Work 阶段。
</HARD-GATE>

### Phase 2: Work

**输入**: `plan/` 中的实现方案
**约束**: 遵循 `context/` 中的开发规范

操作步骤：
1. 按照 plan 逐步执行代码编写
2. 遵循 `context/spec.md` 中的代码质量标准
3. 遵循 `context/arc.md` 中的架构约定
4. 每完成一个步骤，对照 plan 中的验收标准自检
5. 遵循 TDD 纪律：先写失败测试 → 确认测试失败 → 写最小实现 → 确认测试通过

#### 3 次修复升级规则

如果同一个问题连续修复 3 次仍然失败：
1. **停止继续尝试修复** — 这不是"假设失败"，这是"方案错误"
2. **回退到最后一个已知正确状态**
3. **向用户报告**：已尝试的 3 种方案及各自失败原因
4. **讨论是否需要重新设计** — 可能是架构层面的问题，需要回到 Plan 阶段

<HARD-GATE>
Work 阶段的每个步骤完成后，必须运行验证命令并确认通过。
不得在未验证当前步骤的情况下开始下一个步骤。
"代码看起来是对的"不是验证——运行测试、看到绿色输出才是验证。
</HARD-GATE>

### Phase 3: Review

**输入**: Work 阶段产出的代码变更
**标准**: `context/spec.md` 中的质量要求

检查清单：
1. 代码质量: 是否符合项目规范
2. 测试覆盖: 关键路径是否有测试
3. 架构一致性: 是否遵循 arc.md 的设计
4. 约束检查: 是否违反 constraint.md 中的限制
5. 无过度工程: 只实现需求要求的，不添加额外功能

#### 完成前验证规则（Verification Before Completion）

**铁律：没有新鲜验证证据，不得声称完成。**

禁止用语（在未实际验证时）：
- "应该可以工作" / "应该没问题"
- "看起来是对的" / "逻辑上是正确的"
- "测试应该能过"
- 任何在验证之前表达满意或完成的语句

正确做法：
1. 运行全部相关测试 → 粘贴实际输出
2. 如有 lint/fmt 要求 → 运行并粘贴输出
3. 基于实际输出判断是否通过
4. 只有在所有验证通过后才可声称 Review 完成

<HARD-GATE>
Review 发现任何阻断性问题（测试失败、架构违规、安全漏洞），
必须回到 Work 阶段修复。不得带着已知问题进入 Compound 阶段。
"已知问题留到下个迭代修"不是可接受的理由。
</HARD-GATE>

### Phase 4: Compound

**输入**: 整个开发过程中的经验
**输出**: `exp/<feature-name>/` 中的经验链记录 + 可能的 context 更新

操作步骤：
1. 总结本次开发中的关键决策和原因
2. 记录踩过的坑和解决方案（特别是触发了"3次修复升级"的问题）
3. 按经验链规则写入 `exp/`（见下方规则）
4. 如果发现 spec/arc/constraint 需要更新，提出修改建议
5. 将完成的 feature 从 `features/active/` 移到 `features/done/`
6. 开始下一个 feature 的 Plan 阶段

#### 经验链规则

经验不是孤立文档，而是按 feature 形成版本链，记录决策演变：

```
exp/
├── auth/
│   ├── v1.md      ← 初版登录实现经验
│   ├── v2.md      ← 加入 OAuth 时的经验（parent: v1）
│   └── v3.md      ← 修复会话泄漏（parent: v2）
├── search/
│   ├── v1.md      ← 全文搜索初版
│   └── v2.md      ← 性能优化（parent: v1）
└── README.md
```

**经验文件格式：**

```markdown
# <feature-name> v<N>: <简短标题>

**Parent:** v<N-1>（首版写 "无"）
**Feature:** <对应的 feature 文件路径>
**日期:** <YYYY-MM-DD>
**类型:** initial | iteration | fix | optimization

## 本次做了什么决策、为什么

<决策内容>

## 踩过的坑和解决方案

<问题和方案>

## 与上一版经验的关系

<说明本次经验如何延续/修正/推翻了前一版的结论>
（首版写：这是该 feature 的首次经验记录）

## 下次遇到类似情况的建议

<建议>
```

**核心规则：**
- **一个 feature 一个子目录**：目录名与 feature 名保持一致
- **必须引用 parent**：非首版经验必须在 Parent 字段标注前一版本号，并在"与上一版经验的关系"中说明演变
- **只追加不修改**：已有的经验文件不可修改，新经验只能作为新版本追加
- **跨 feature 引用**：如果本次经验涉及其他 feature 的经验，用 `参见 exp/<other-feature>/v<N>.md` 引用

<HARD-GATE>
经验总结不得跳过。"这次没什么特别的"不是跳过的理由。
每次开发必须至少记录：做了什么决策、为什么这样决策、
下次遇到类似情况的建议。哪怕只有三句话，也必须写入 exp/。
</HARD-GATE>

## 使用流程

### 初始化项目

当用户要求初始化项目开发流程时：

1. 创建 `.ai-workflow/` 目录结构
2. 根据项目情况生成初始 context 文件：
   - `spec.md`: 从项目现有代码/文档推断开发规范
   - `arc.md`: 从项目结构推断架构方案
   - `constraint.md`: 从项目依赖和配置推断约束
3. 将 `references/` 中的模板直接复制到对应目录，生成 README.md：
   - `requirements.md` → `.ai-workflow/requirements/README.md`
   - `context.md` → `.ai-workflow/context/README.md`
   - `features-active.md` → `.ai-workflow/features/active/README.md`
   - `plan.md` → `.ai-workflow/plan/README.md`
   - `exp.md` → `.ai-workflow/exp/README.md`
4. 除上述 reference 文件明确对应的 README.md 外，不额外创建模板文件

### 需求分析

当用户提供原始需求时：

1. 将原始需求保存到 `requirements/`
2. 分析需求，拆分为独立的可执行功能
3. 为每个功能生成 feature 文件，放入 `features/active/`
4. Feature 文件应包含：背景、目标、验收标准、依赖关系、优先级

### 开始开发

当用户要求开发某个 feature 时：

1. 读取该 feature 文件 + 全部 context 文件
2. 进入 Plan → Work → Review → Compound 循环
3. 使用 TodoWrite 跟踪 plan 中每个步骤的进度
4. 完成后归档到 `features/done/`

## 子 Agent 调度指南

当任务足够复杂时，可以将工作分派给子 Agent。遵循以下原则：

### 何时使用子 Agent

- 计划包含 5+ 个独立步骤
- 存在可并行的独立任务（如：多个模块互不依赖）
- 需要专项审查（实现 Agent + 审查 Agent 角色分离）

### 调度规则

1. **上下文隔离**：子 Agent **永远不继承主会话上下文**。控制器必须将完整任务文本直接注入子 Agent prompt，而不是让子 Agent 去读文件
2. **完整注入**：子 Agent 的 prompt 必须包含：任务描述、相关代码上下文、验收标准、约束条件
3. **状态报告**：子 Agent 完成后必须报告四种状态之一：
   - `DONE` — 任务完成，所有验证通过
   - `DONE_WITH_CONCERNS` — 任务完成但有潜在风险需关注
   - `BLOCKED` — 遇到阻塞，需要额外信息或决策
   - `NEEDS_CONTEXT` — 注入的上下文不足，需要补充
4. **不信任报告**：子 Agent 说 "完成了" 不代表真的完成。控制器（或审查 Agent）必须独立验证代码
5. **并行调度**：独立任务应并行分派，但有依赖关系的任务必须串行

### 角色分离模式

```
控制器（主会话）
    │
    ├── 实现 Agent（per task）── 写代码、写测试
    │       │
    │       ▼
    ├── Spec 审查 Agent ─────── 实现是否符合需求规格
    │       │
    │       ▼
    └── 质量审查 Agent ─────── 代码质量、架构一致性、安全性
```

审查 Agent 的 prompt 应包含："实现者的报告可能不完整、不准确或过于乐观。你必须独立验证一切。"

## Agent 常见逃避模式（反合理化表）

<EXTREMELY-IMPORTANT>
以下是 AI Agent 在执行此工作流时会使用的常见逃避借口。
如果你发现自己正在产生类似想法，这是你试图违反流程的信号——停下来，严格遵循流程。
</EXTREMELY-IMPORTANT>

### Plan 阶段逃避

| 合理化借口 | 为什么是错的 |
|-----------|-------------|
| "这个改动太小了不需要计划" | 小改动引发的回归 bug 比大改动更多，正因为小才容易忽略边界 |
| "我已经知道怎么做了，直接写更快" | 知道怎么做 ≠ 考虑了所有边界情况和副作用 |
| "用户赶时间，计划太慢了" | 没有计划导致的返工比写计划花的时间多 3-5 倍 |
| "先出个框架，细节后面补" | 没有细节的框架不是计划，是愿望 |

### Work 阶段逃避

| 合理化借口 | 为什么是错的 |
|-----------|-------------|
| "先写代码再补测试，效率更高" | 事后补测试只验证实现，不验证需求。先测试才能保证需求覆盖 |
| "这段代码逻辑很简单，不需要测试" | "简单"是主观判断。如果真的简单，写测试也只需要 30 秒 |
| "这个步骤的验证显而易见，不用运行" | 显而易见的正确是 bug 的温床——运行验证才是唯一证据 |
| "我改动很小，不会影响其他部分" | 最危险的 bug 来自"不会影响到"的假设 |

### Review 阶段逃避

| 合理化借口 | 为什么是错的 |
|-----------|-------------|
| "我写的时候已经检查过了" | 实现者审查自己的代码 ≠ Review，你有盲区 |
| "测试都过了，Review 走个形式就行" | 测试只验证功能正确，Review 还要检查架构、安全、可维护性 |
| "这个问题不严重，下个迭代再修" | "下个迭代"是 bug 的黑洞——进去就出不来了 |

### Compound 阶段逃避

| 合理化借口 | 为什么是错的 |
|-----------|-------------|
| "这次没什么特别的，不用总结" | 每次开发都有决策，没有"特别的"意味着你没有反思 |
| "经验总结以后再写" | 记忆衰减，当下不写 24 小时后就忘了 80% |
| "代码就是最好的文档" | 代码记录了"做了什么"，不记录"为什么这样做"和"考虑过什么替代方案" |

## 工作原则

- **MVP First**: 先实现最小可用版本，再迭代优化
- **Context 先行**: 每次开发前先读取 context/ 了解约束
- **经验驱动**: 每次开发后在 exp/<feature>/ 追加经验链，Plan 阶段沿链追溯避免重复犯错
- **渐进归档**: feature 完成一个归档一个，保持 active/ 精简（2-5 个）
- **模板规范**: 每个目录的 README.md 提供详细的文件格式模板
- **验证驱动**: 没有运行验证的代码一律视为未完成
- **3 次即止**: 同一问题修 3 次不通就升级讨论，不要死磕
- **不信任自己**: 对自己的判断保持怀疑，用证据（测试输出）说话
