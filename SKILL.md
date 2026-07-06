---
name: "doc-driven"
description: "Use when user says doc-driven, starts a project intake, describes a feature idea, updates requirements, consolidates docs, or asks to execute an existing requirement."
---

# Doc-Driven

文档驱动项目：维护好一份文档就是维护好一个项目。作为 AI Coding Agent 的自然语言入口层，自动完成意图识别、文档路由、质量门禁、冲突裁决和文档收敛——用户无需理解文档结构，Agent 无需猜该写哪里。

## 使用前说明

- 本 Skill 使用 `templates/` 子目录下的模板文件（`requirement.md`、`requirements-index.md`、`requirement-quality-gate.md`），路径相对于本 SKILL.md 所在目录。
- 文档中 "Superpowers" 指代「需求 → 执行计划 → 执行规格 → 验证证据」的完整工作流。如果你的平台没有内置 Superpowers 能力，可以将 plan/spec/test 视为普通文档，由 Agent 按模板手动生成。
- 文档中引用的项目文件（`AGENTS.md`、`CLAUDE.md`、`docs/system/*.md` 等）均按项目实际存在情况读取，缺失时跳过不影响核心流程。
- 本 Skill 为平台无关设计，可在 Claude Code、Cursor、Trae、Codex 等任意 AI Coding Agent 中使用。

## Core Rule

The user should not need to understand docs structure or skill names. If the user says `doc-driven 初始化` or `Use Skill: doc-driven ...`, perform intent recognition and route the work automatically. Ask the user only for blocking missing information.

## Three-Step Usage Model

### Step 1: Initialization

Trigger phrases:

- `doc-driven 初始化`
- `Use Skill: doc-driven 初始化`
- `初始化项目需求文档`
- `读取项目并建立需求体系`

Behavior:

1. Read available conversation context for user goals and durable requirements.
2. Read project files and existing docs:
   - `README.md`
   - `AGENTS.md`
   - `CLAUDE.md`
   - `docs/system/project-knowledge-system.md`
   - `docs/system/current-architecture.md`
   - `docs/system/product-rules.md`
   - `docs/system/business-and-competitive-strategy.md`
   - `docs/system/data-model.md`
   - `docs/system/api-inventory.md`
   - `docs/system/known-pitfalls.md`
   - existing `docs/requirements/`, `docs/design/`, `docs/promo/`, `docs/superpowers/`
3. Create missing directories:
   - `docs/requirements/`
   - `docs/superpowers/plans/`
   - `docs/superpowers/specs/`
   - `docs/superpowers/tests/`
   - `docs/design/`
   - `docs/promo/`
   - `docs/requirements/archive/`
4. Create missing baseline docs from confirmed facts only. Mark unknowns as `待验证`.
5. Create or update `docs/requirements/index.md` as the requirement map.
6. Report what was created, what was updated, and what still needs user confirmation.

Initialization is allowed to read broadly. It must not invent facts or implementation details.

### Step 2: Daily Auto Routing

After initialization, users can speak naturally:

- `我想做一个功能...`
- `把这个需求改一下...`
- `这个需求可以开始做了`
- `帮我设计一下这个页面`
- `调研一下竞品`
- `收尾同步文档`
- `整理文档` / `收敛文档` / `压缩文档`

They can also explicitly write:

```text
Use Skill: doc-driven <想要做的事情>
```

The skill must classify the intent and route it:

| User intent | Destination |
| --- | --- |
| New feature idea, user need, product problem | `docs/requirements/YYYY-MM-DD-<topic>.md` |
| Requirement update or clarification | Update existing `docs/requirements/*.md` |
| User says execute / start / implement existing requirement | 如果就绪，进入 plan/spec 工作流；否则更新需求并提阻塞问题 |
| Page style, interaction, visual reference | `docs/design/` |
| Research, competitors, market, product strategy | `docs/system/business-and-competitive-strategy.md` or `docs/requirements/` depending on durability |
| Architecture, API, data model, known pitfalls | `docs/system/`（通过读取现有系统文档来更新） |
| Promo, poster, video, publish copy | `docs/promo/` |
| 整理文档 / 收敛文档 / 压缩文档 / 文档去重 / 归档 | Execute Step 3 文档收敛 workflow |
| Verification output, test result, training evidence | `docs/superpowers/tests/` |

Default behavior: choose the best route silently, then tell the user where the artifact was written.

### Step 3: 文档收敛

触发词：

- `整理文档` / `收敛文档` / `压缩文档` / `文档去重` / `归档`

目标：

- 一个主题 = 一份权威事实源（canonical document）
- 消除冗余、融合相似内容、归档历史文档
- 降低 AI 读取 prompt 消耗，同时保证搜索可达性和事实完整性

> 注意：`docs/business-research/` 不参与收敛，由用户自行管理。

#### 3.1 收敛范围

只维护以下目录：

```
docs/requirements/       # canonical 需求文档（合并冲突分析）
  archive/               # 已归档历史需求
docs/superpowers/
  plans/                 # 执行计划
  specs/                 # 执行规格
  tests/                 # 验证证据
docs/system/             # 系统长期事实
```

不动：`docs/business-research/`、`docs/promo/`、`docs/design/`

#### 3.2 收敛流程

**阶段 A：消除冗余 change-spec**

如果项目中存在独立的 change-spec / 冲突分析文档（如 `docs/system/changes/` 目录下的文件）：
- 对每个 change-spec，查找同名同主题的 requirement
  - 找到 → 把冲突分析、影响范围合并到 requirement，原文件标记 `superseded_by` 后删除
  - 找不到 → 提升为 requirement 文件，移入 `docs/requirements/`
- 清理完成后，删除空的 change-spec 目录和相关模板文件

**阶段 B：消除 session-summary**

对 `docs/system/session-summary-*.md`、`docs/system/session-lessons-*.md`、`docs/requirements/current-session-summary-*.md` 等文件，按内容类型分流：

| 内容类型 | 目标位置 | 操作 |
| --- | --- | --- |
| 踩坑/根因/禁止做法/正确做法/检查方式 | `docs/system/known-pitfalls.md` | 追加到对应章节，去重 |
| 模块/入口/数据路径的当前事实变化 | `docs/system/current-architecture.md` | 更新对应章节 |
| 产品交互/规则变化 | `docs/system/product-rules.md` | 更新对应章节 |
| 字段/状态机语义变化 | `docs/system/data-model.md` | 更新对应章节 |
| 接口增删改 | `docs/system/api-inventory.md` | 更新对应章节 |
| 需求级新功能讨论 | `docs/requirements/` | 创建或更新对应需求文件 |
| 纯粹"今天聊了什么"的流水账 | 丢弃 | 不保留 |

分流完成后删除原 session-summary 文件。

**阶段 C：test-report 瘦身**

对 `docs/superpowers/tests/` 下每个文件：

- 去掉"本轮目标""背景描述"段（这些在 requirement 中已有）
- 只保留：测试命令、关键结果、指标数据、失败原因
- 加一行 backlink 指向 canonical requirement

**阶段 D：需求去重 + 归档 + 重建索引**

1. 同一 topic cluster 内多份 requirement → 合并为一份 canonical，其余标记 `superseded_by: <canonical_path>`
2. 状态 `Implemented` 且事实已完全被 system 文档吸收的 → 移到 `docs/requirements/archive/YYYY-MM/`，原路径保留索引条目
3. 重建 `docs/requirements/index.md`（去掉日期分段，统一为表格索引）
4. 更新 `docs/system/changelog.md` 记录本次收敛操作

#### 3.3 冲突裁决规则

```
发现事实矛盾 → 查源码/配置文件确认 →
  ├─ 源码明确 → 以源码为准，自动修正文档
  ├─ 源码也模糊 → 两条都保留，标记 待验证
  └─ 完全无法判断 → 提一个精简问题给用户（只说矛盾点和两种可能，不输出全文）
```

提给用户的问题格式：

```text
[冲突] <主题>：A 文档说 <事实A>，B 文档说 <事实B>。源码中未找到明确结论。请确认哪个是正确的？
```

#### 3.4 质量自检（内部执行，不输出报告）

收敛完成后，skill 内部自检两项：

1. **搜索可达性**：任意功能关键词搜索后，能否在 ≤2 步内到达 canonical 文档
2. **事实完整性**：被删除/归档的旧文档中，每条唯一事实在 canonical 中是否仍有对应

自检不通过时，回退修正，不输出报告。收敛完成只告诉用户一句：`已收敛 N 份文档，当前活跃文档 M 份。`

## Inputs

Read before writing:

- `README.md`
- `AGENTS.md`
- `CLAUDE.md`
- `docs/system/project-knowledge-system.md`
- Relevant `docs/system/current-architecture.md`, `product-rules.md`, `data-model.md`, `api-inventory.md`, `known-pitfalls.md`
- Related files under `docs/requirements/`

If implementation is likely, search related code/tests with `rg` before claiming feasibility.

## Output Location

Save requirements to:

```text
docs/requirements/YYYY-MM-DD-<short-topic>.md
```

If the project already has a topic naming convention, follow it. Use `templates/requirement.md` as the base template.

Save the requirement map to:

```text
docs/requirements/index.md
```

Use `templates/requirements-index.md` as the base template.

## What Belongs in Requirements

Write:

- User original intent and source summary.
- Business background, user type, scenario, pain point.
- Desired outcomes in product language.
- Acceptance criteria that can be verified.
- Inputs, outputs, examples, and counterexamples.
- Constraints: platform, privacy, security, performance, cost, timeline, compatibility.
- Non-goals and explicitly excluded scope.
- Edge cases and negative scenarios.
- Open questions and decisions needed before planning.
- Links to affected system facts and existing requirements.

## What Does Not Belong

Do not write:

- Detailed implementation tasks.
- Code snippets intended for implementation.
- Database fields or API contracts unless they already exist and are referenced as current facts.
- UI visual design details; put those in `docs/design/`.
- Promo copy or publishing assets; put those in `docs/promo/`.
- Test execution logs; put those in `docs/superpowers/tests/`.
- Unverified competitor claims.
- Secrets, tokens, private user material, raw personal data.
- Assumptions presented as facts.

If technical feasibility matters, record it as a question or constraint, then hand off to plan/spec.

## Requirement Quality Bar

A requirement is ready for plan/spec only when it answers:

- Who is the user or actor?
- What outcome do they need?
- Why does this matter now?
- What is the current system behavior or gap?
- What must be true for acceptance?
- What is explicitly out of scope?
- Which system facts constrain the solution?
- Which known pitfalls must be avoided?
- What remains unresolved?

If any answer is unknown, write `待验证` or ask the user. Do not guess.

## Acceptance Criteria Style

Prefer testable statements:

- `WHEN <condition/action>, THEN <observable result>.`
- `GIVEN <state>, WHEN <action>, THEN <result>.`
- `The system SHALL <behavior> when <condition>.`

Good:

- `GIVEN a completed SfM task with ROI selected, WHEN local training starts, THEN training input excludes pixels outside the ROI-derived crop area.`
- `WHEN camera permission is denied, THEN the app shows a recoverable permission prompt and does not clear the current task.`

Bad:

- `ROI should work better.`
- `Make the UI smoother.`
- `Optimize training as much as possible.`

## Superpowers Handoff (Plan/Spec 交接)

需求通过质量门禁后，进入 plan/spec 工作流（本文统称 "Superpowers"）：

1. 多步骤工作 → 生成执行计划。
2. 执行计划保存到：

```text
docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md
```

3. 正式规格保存到：

```text
docs/superpowers/specs/YYYY-MM-DD-<feature-name>.md
```

4. 实现完成后，验证证据保存到：

```text
docs/superpowers/tests/YYYY-MM-DD-<feature-name>.md
```

> 注：如果你的 Agent 平台没有内置 Superpowers 能力，可将 plan/spec/test 作为普通 Markdown 文档，由 Agent 手动生成。需求文档是 plan 的唯一事实源。如果计划过程中发现新的产品范围或矛盾，先更新需求再继续实现。

## Execution Intent

当用户说 `执行`、`开始做`、`实现`、`按这个做` 或类似词语时：

1. 从当前对话或 `docs/requirements/index.md` 定位目标需求。
2. 多个需求匹配时，请用户选择。
3. 无对应需求时，先创建。
4. 需求有未解决的阻塞问题时，只提这些问题。
5. 就绪后，进入 plan/spec 工作流：
   - 多步骤工作 → `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
   - 需要正式规格 → `docs/superpowers/specs/YYYY-MM-DD-<feature-name>.md`
6. 需求和 plan/spec 就绪前，不直接开始实现。

## Process

1. Classify the request: initialization, new requirement, requirement update, execution intent, design doc, research, system sync, promo, verification, consolidation, or audit.
2. Read system facts and related requirements.
3. Extract user intent and constraints.
4. Separate facts, assumptions, open questions, and implementation ideas.
5. Select the right template or target directory.
6. Draft or update the artifact.
7. Run `templates/requirement-quality-gate.md` for requirement artifacts.
8. 如果就绪且用户意图是执行，进入 plan/spec 工作流。
9. Update `docs/system/changelog.md` when the requirement changes durable product direction or project knowledge structure.

## Common Mistakes

- Turning requirements into a technical design too early.
- Losing the user's original wording and replacing it with generic PRD prose.
- Writing acceptance criteria that cannot be tested.
- Hiding uncertainty instead of marking `待验证`.
- Duplicating system facts instead of linking to `docs/system/`.
- 在非目标、约束和验收标准明确之前就开始 plan/spec。
- Forcing the user to name internal docs or skills after initialization.
- Treating `Use Skill: doc-driven <task>` as only requirement capture; it must classify the task first.