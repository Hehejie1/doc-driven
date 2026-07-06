---
name: "doc-driven"
description: "Use when user says doc-driven, starts a project intake, describes a feature idea, updates requirements, consolidates docs, or asks to execute an existing requirement."
---

# Doc-Driven

文档驱动项目：维护好一份文档就是维护好一个项目。三大核心功能：项目初始化、需求驱动执行、对话总结与收敛。

## 使用前说明

- 本 Skill 使用 `templates/` 子目录下的模板文件，路径相对于本 SKILL.md 所在目录。
- "Superpowers" 指代「需求 → 执行计划 → 执行规格 → 验证证据」的完整工作流。平台无此能力时，将 plan/spec/test 视为普通 Markdown 手动生成即可。
- 文档中引用的项目文件均按实际存在情况读取，缺失时跳过或按模板生成。
- 本 Skill 平台无关，可在 Claude Code、Cursor、Trae、Codex 等任意 AI Coding Agent 中使用。

## 功能一：项目初始化

### 触发词

- `doc-driven 初始化`
- `Use Skill: doc-driven 初始化`
- `初始化项目`
- `初始化项目文档`

### 行为

#### 1.1 创建 AGENTS.md 与 CLAUDE.md（如不存在）

**AGENTS.md** — Agent 行为法。内容必须包含：
- 开工必读文档列表（README.md、CLAUDE.md、docs/system/*.md）
- 禁止行为（不删用户代码、不猜事实、不做无关重构、不跳过验证等）
- 验收口径（任何阶段必须给出可验证产物）
- 新需求开工前必答的 8 个问题（影响哪些模块、是否冲突、最小改动、回归测试等）

**CLAUDE.md** — LLM 执行纪律。内容必须包含：
- 先思考再编码（不假设、不隐藏困惑、暴露权衡）
- 简洁优先（不写未要求的代码、不抽象单次使用的逻辑）
- 手术式修改（不改相邻代码、不改格式、只删自己造成的死代码）
- 目标驱动执行（定义成功标准、循环直到验证通过）

> 模板参考本 Skill 的 `templates/agents-template.md` 和 `templates/claude-template.md`。若项目已有则保留，仅在明显缺失关键内容时提示补充。

#### 1.2 创建 docs/ 目录结构

```
docs/
├── requirements/              # 需求文档
│   └── archive/               # 已归档需求
├── superpowers/
│   ├── plans/                 # 执行计划
│   ├── specs/                 # 执行规格
│   └── tests/                 # 验证证据
├── system/                    # 系统长期事实
├── design/                    # UI 设计
└── promo/                     # 宣传物料
```

#### 1.3 扫描现有代码

如果项目已有代码，执行：
1. 遍历项目目录结构，识别主要模块和入口文件
2. 识别技术栈（语言、框架、数据库、关键依赖）
3. 识别数据模型（字段、状态机、缓存策略）
4. 识别接口（HTTP/JNI/CLI/本地协议）
5. 将识别结果写入 `docs/system/`：
   - `current-architecture.md`：模块边界、入口、运行方式
   - `data-model.md`：字段语义、状态流转
   - `api-inventory.md`：接口清单、调用方、错误码

标记不确定项为 `待验证`，不编造事实。

#### 1.4 创建需求索引

生成 `docs/requirements/index.md`，使用 `templates/requirements-index.md` 作为模板。

#### 1.5 汇报

报告创建和更新的文件清单，列出需要用户确认的 `待验证` 项。

---

## 功能二：需求驱动执行

### 触发词

- `我想做一个功能...`
- `把这个需求改一下...`
- `这个需求可以开始做了` / `执行` / `开始做` / `实现`
- `帮我设计一下这个页面`
- `调研一下竞品`

### 2.1 意图路由

用户自然语言 → 自动分类 → 写入正确位置：

| 用户意图 | 落点 |
| --- | --- |
| 新功能想法、用户痛点、业务需求 | `docs/requirements/YYYY-MM-DD-<topic>.md` |
| 修改已有需求 | 更新对应需求文件 |
| 执行/实现已有需求 | 质量门禁 → plan → spec → 实现 |
| 页面设计、交互、视觉风格 | `docs/design/` |
| 竞品调研、产品策略 | `docs/system/business-and-competitive-strategy.md` |
| 宣传物料、发布文案 | `docs/promo/` |

### 2.2 创建需求

使用 `templates/requirement.md` 作为模板，写入 `docs/requirements/YYYY-MM-DD-<topic>.md`。

需求文档必须包含：原始意图、背景痛点、目标结果、非目标、当前系统事实、用户故事、验收标准、输入/输出样例、约束条件、边界场景、开放问题、与系统事实的冲突检查。

### 2.3 质量门禁

需求进入 plan/spec 前，运行 `templates/requirement-quality-gate.md` 检查：

1. 用户是谁？需要什么结果？为什么现在做？
2. 当前系统行为/差距是什么？
3. 验收标准可验证吗？（GIVEN/WHEN/THEN 或 SHALL）
4. 明确不做什么？受哪些约束？
5. 与 product-rules.md、data-model.md、api-inventory.md 有冲突吗？
6. 是否会重新引入 known-pitfalls.md 中的问题？
7. 是否包含正常路径和失败/边界场景？
8. 开放问题是否只保留会阻塞 plan/spec 的？
9. 是否泄露了密钥、token 或隐私数据？

答不上来标 `待验证`，不猜。只通过全部检查的需求才进入 plan/spec。

### 2.4 生成执行计划（Plan）

需求达到 Ready for Plan 后，生成执行计划：

```text
docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md
```

Plan 必须包含：任务拆分、依赖顺序、涉及文件、验收标准映射、风险点、回滚方案。

### 2.5 生成执行规格（Spec）

Plan 确认后，生成正式规格：

```text
docs/superpowers/specs/YYYY-MM-DD-<feature-name>.md
```

Spec 必须包含：具体文件路径、接口变更、数据结构变更、测试命令、验证方式。

### 2.6 实现与验证

Spec 确认后开始实现。完成后写入验证证据：

```text
docs/superpowers/tests/YYYY-MM-DD-<feature-name>.md
```

Test 只保留：测试命令、关键结果、指标数据、失败原因。不重复需求背景和目标。加一行 backlink 指向 canonical requirement。

### 2.7 写回规则

实现完成后，检查是否改变了持久化的项目知识，按以下规则写回：

| 变更类型 | 写回目标 |
| --- | --- |
| 架构或模块边界变化 | `docs/system/current-architecture.md` |
| 产品目标、规则、用户确认交互变化 | `docs/system/product-rules.md` |
| 商业模式、竞品定位变化 | `docs/system/business-and-competitive-strategy.md` |
| 字段语义、状态机、缓存策略变化 | `docs/system/data-model.md` |
| HTTP/JNI/CLI/本地协议变化 | `docs/system/api-inventory.md` |
| 发现新的反复性失败、根因、禁止做法 | `docs/system/known-pitfalls.md` |
| 重要系统演进 | `docs/system/changelog.md` |
| UI 风格、页面结构、组件规范变化 | `docs/design/` |
| 宣传策略、发布物料变化 | `docs/promo/` |

---

## 功能三：对话总结与收敛

### 触发词

- `整理文档` / `收敛文档` / `压缩文档` / `文档去重` / `归档`
- `总结对话` / `沉淀经验` / `收尾同步文档`

### 3.1 对话经验沉淀

分析当前对话中的所有信息，提取以下内容并写入对应文档：

| 提取内容 | 写入位置 | 操作 |
| --- | --- | --- |
| 踩坑/根因/禁止做法/正确做法/检查方式 | `docs/system/known-pitfalls.md` | 追加到对应章节，去重 |
| 模块/入口/数据路径的变化 | `docs/system/current-architecture.md` | 更新对应章节 |
| 产品交互/规则变化 | `docs/system/product-rules.md` | 更新对应章节 |
| 字段/状态机语义变化 | `docs/system/data-model.md` | 更新对应章节 |
| 接口增删改 | `docs/system/api-inventory.md` | 更新对应章节 |
| 需求级新功能讨论 | `docs/requirements/` | 创建或更新对应需求文件 |
| 纯粹"今天聊了什么"的流水账 | 丢弃 | 不保留 |

### 3.2 文档收敛

对以下目录执行收敛，保证一个主题 = 一份权威事实源：

收敛范围：
```
docs/requirements/       # canonical 需求文档
  archive/               # 已归档历史需求
docs/superpowers/
  plans/                 # 执行计划
  specs/                 # 执行规格
  tests/                 # 验证证据
docs/system/             # 系统长期事实
```

不参与收敛：`docs/business-research/`、`docs/promo/`、`docs/design/`

#### 阶段 A：消除冗余 change-spec

如果存在独立的 change-spec / 冲突分析文档（如 `docs/system/changes/` 下的文件）：
- 查找同名同主题的 requirement → 合并冲突分析和影响范围 → 原文件标记 `superseded_by` 后删除
- 找不到对应 requirement → 提升为 requirement 文件，移入 `docs/requirements/`

#### 阶段 B：test-report 瘦身

对 `docs/superpowers/tests/` 下每个文件：
- 去掉"本轮目标""背景描述"段（已在 requirement 中）
- 只保留：测试命令、关键结果、指标数据、失败原因
- 加一行 backlink 指向 canonical requirement

#### 阶段 C：需求去重 + 归档 + 重建索引

1. 同一 topic 内多份 requirement → 合并为一份 canonical，其余标记 `superseded_by`
2. 状态 `Implemented` 且事实已完全被 system 文档吸收 → 移到 `archive/YYYY-MM/`
3. 重建 `docs/requirements/index.md`（统一为表格索引）
4. 更新 `docs/system/changelog.md` 记录本次收敛

### 3.3 冲突裁决

```
发现事实矛盾 → 查源码/配置文件确认 →
  ├─ 源码明确 → 以源码为准，自动修正文档
  ├─ 源码也模糊 → 两条都保留，标记 待验证
  └─ 完全无法判断 → 精简提问（只说矛盾点和两种可能）
```

提问格式：`[冲突] <主题>：A 文档说 <事实A>，B 文档说 <事实B>。源码中未找到明确结论。请确认？`

### 3.4 质量自检（内部执行，不输出报告）

1. **搜索可达性**：任意功能关键词能否在 ≤2 步内到达 canonical 文档
2. **事实完整性**：被删除/归档的旧文档中，每条唯一事实在 canonical 中是否仍有对应

收敛完成只报告：`已收敛 N 份文档，当前活跃文档 M 份。`

---

## 模板文件

| 模板 | 用途 | 使用场景 |
| --- | --- | --- |
| `templates/requirement.md` | 需求文档模板 | 功能二：创建需求 |
| `templates/requirements-index.md` | 需求索引模板 | 功能一：初始化 / 功能三：重建索引 |
| `templates/requirement-quality-gate.md` | 质量门禁检查清单 | 功能二：质量门禁 |
| `templates/agents-template.md` | AGENTS.md 模板 | 功能一：项目初始化 |
| `templates/claude-template.md` | CLAUDE.md 模板 | 功能一：项目初始化 |

## 验收标准风格

优先可验证表达：

- `GIVEN <state>, WHEN <action>, THEN <observable result>.`
- `WHEN <condition>, THEN <observable result>.`
- `The system SHALL <behavior> when <condition>.`

Good：`WHEN camera permission is denied, THEN the app shows a recoverable permission prompt and does not clear the current task.`

Bad：`Make the UI smoother.` / `Optimize training as much as possible.`

## 旧项目接入

1. 盘点现有需求、架构、产品策略、数据、接口、UI 设计、宣传素材、已知失败
2. 按匹配层级创建最小文档，只写已确认事实
3. 产品策略和商业判断放入 `docs/system/`，不放在营销目录
4. 未知项列为显式验证任务
5. 接入期间不实现无关功能

## 常见错误

- 需求文档写成实施方案
- 用"优化、提升、完善"等词但没有可观察验收
- 只写 happy path，不写失败或边界情况
- 隐藏不确定性而不是标记 `待验证`
- 复述系统事实而不是链接到 `docs/system/`
- 在非目标、约束和验收标准明确之前就开始 plan/spec
- 改完代码不写回对应的 system/ 文档
- 把 `Use Skill: doc-driven <task>` 只当需求捕获用，必须按意图分类路由