# Doc-Driven

> 文档驱动项目：维护好一份文档就是维护好一个项目。

AI Coding Agent 的**自然语言文档入口层**。让用户用日常语言描述需求，Agent 自动完成意图识别、文档路由、质量门禁、冲突裁决和文档收敛——无需理解文档结构或 Skill 名称。

## 解决的问题

用 AI Agent 写代码时，最大的瓶颈往往不是"Agent 写不好代码"，而是：

| 痛点 | Doc-Driven 的解法 |
| --- | --- |
| 需求散落在聊天记录里，换会话就丢失 | 自动写入 `docs/requirements/`，结构化持久化 |
| 需求越积越多，重复、冲突、版本混乱 | 文档收敛流程：去重、冲突裁决、归档、重建索引 |
| 需求直接跳实现，缺少质量把关 | 9 题质量门禁，答不上来就标 `待验证`，不猜 |
| Agent 分不清需求、设计、调研该写哪 | 意图分类路由表，自动写入正确目录 |
| 项目文档膨胀，Agent 上下文越读越长 | 一个主题 = 一份权威事实源，定期瘦身 |

## 快速开始

### 1. 初始化

将本目录放入项目的 `.trae/skills/doc-driven/`（或其他 Agent 平台的 Skill 目录），然后在对话中说：

```
doc-driven 初始化
```

Agent 会自动：
- 读取项目 `README.md`、`AGENTS.md`、`CLAUDE.md` 和 `docs/system/` 下的现有文档
- 创建缺失的目录结构（`docs/requirements/`、`docs/superpowers/`、`docs/design/`、`docs/promo/`）
- 基于已有事实生成基线文档，未知项标记 `待验证`
- 创建 `docs/requirements/index.md` 作为需求索引

### 2. 日常使用

初始化后，不需要说 Skill 名称，直接用自然语言：

```
我想做一个多端同步功能，手机拍完照片自动传到电脑上
```

```
这个需求不用做视频导入，先只支持照片
```

```
这个需求可以开始做了
```

```
帮我设计一下任务详情页
```

```
整理文档
```

Agent 自动识别意图并路由到正确位置。

### 3. 执行需求

```
这个需求可以开始做了
```

Agent 会：
1. 检查需求是否达到 Ready for Plan 状态
2. 通过质量门禁后，生成 plan/spec
3. plan/spec 就绪后才开始实现

## 核心方法论

### 意图路由

用户的每一句话都被分类并路由到正确的目标：

| 用户意图 | 落点 |
| --- | --- |
| 新功能想法、用户痛点 | `docs/requirements/YYYY-MM-DD-<topic>.md` |
| 修改已有需求 | 更新对应文件 |
| 执行/实现 | 检查质量门禁 → plan/spec → 实现 |
| 页面设计、交互 | `docs/design/` |
| 竞品调研、产品策略 | `docs/system/business-and-competitive-strategy.md` |
| 宣传物料、发布文案 | `docs/promo/` |
| 整理/收敛/归档文档 | 文档收敛流程 |

### 质量门禁

需求进入 plan/spec 前必须通过 9 题检查：

- 用户是谁？需要什么结果？为什么现在做？
- 当前系统行为/差距是什么？
- 验收标准可验证吗？
- 明确不做什么？受哪些约束？
- 与现有系统事实有冲突吗？
- 哪些已知坑必须避免？
- 还有什么未解决？

答不上来就标 `待验证`，不猜。

### 文档收敛

触发词：`整理文档` / `收敛文档` / `压缩文档`

四个阶段：
- **A** — 消除冗余 change-spec，合并到 canonical requirement
- **B** — 消除 session-summary，按内容分流到 system 文档或丢弃
- **C** — test-report 瘦身，只保留验证证据
- **D** — 需求去重、归档、重建索引

收敛完成：`已收敛 N 份文档，当前活跃文档 M 份。`

## 目录结构

```
doc-driven/
├── SKILL.md                          # Skill 主文件（Agent 执行指令）
├── README.md                         # 本文件（给人看）
└── templates/
    ├── requirement.md                # 需求文档模板
    ├── requirements-index.md         # 需求索引模板
    └── requirement-quality-gate.md   # 质量门禁检查清单
```

初始化后在项目根目录创建的文档结构：

```
docs/
├── requirements/                     # 需求文档
│   ├── index.md                      # 需求索引（入口）
│   ├── YYYY-MM-DD-<topic>.md        # 各需求文件
│   └── archive/                      # 已归档需求
├── superpowers/
│   ├── plans/                        # 执行计划
│   ├── specs/                        # 执行规格
│   └── tests/                        # 验证证据
├── system/                           # 系统长期事实（已有则复用）
├── design/                           # UI 设计文档
└── promo/                            # 宣传物料
```

## 支持的平台

平台无关设计，可在以下任意 AI Coding Agent 中使用：

- **Trae IDE** — 放入 `.trae/skills/doc-driven/`
- **Claude Code** — 通过 `AGENTS.md` 引用或自定义 Skill 加载
- **Cursor** — 通过 `.cursorrules` 引用
- **GitHub Copilot** — 通过 `copilot-instructions.md` 引用
- **Codex / Windsurf** — 放入平台对应的 Skill 目录

如果平台没有内置 Skill 系统，可将 `SKILL.md` 内容作为 Agent 系统提示词的一部分使用。

## 与现有项目文档的兼容

Doc-Driven 设计为渐进式接入：

- 已有 `docs/` 的项目 → 读取现有文档，补充缺失
- 全新项目 → 初始化时创建完整目录结构
- 没有 `docs/system/` → 跳过 system 文档读取，按需逐步建立
- 没有 `AGENTS.md` / `CLAUDE.md` → 跳过，不影响核心流程

## 常见问题

**Q: "Superpowers" 是什么？**
A: 本文中的 "Superpowers" 是「需求 → 执行计划 → 执行规格 → 验证证据」工作流的统称。在 Trae IDE 中这是内置能力，在其他平台可以将 plan/spec/test 视为普通 Markdown 文档手动生成。

**Q: 模板路径怎么找？**
A: 模板文件在 `templates/` 子目录下，路径相对于 `SKILL.md` 所在目录。

**Q: 可以和 `project-knowledge-ledger` 一起用吗？**
A: 可以。两者互补：`doc-driven` 负责文档入口、意图路由和文档收敛，`project-knowledge-ledger` 负责系统事实（架构、数据模型、接口、踩坑）的持续更新。

**Q: 中文项目才能用吗？**
A: 模板和触发词目前是中文，但方法论与语言无关。可以自行翻译模板和触发词为英文或其他语言。

## License

MIT