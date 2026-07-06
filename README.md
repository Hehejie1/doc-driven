# Doc-Driven

> 文档驱动项目：维护好一份文档就是维护好一个项目。

AI Coding Agent 的**文档驱动方法论**。三大核心功能：项目初始化、需求驱动执行、对话总结与收敛。

## 解决的问题

| 痛点 | Doc-Driven 的解法 |
| --- | --- |
| 新项目接入 Agent 无从下手 | **功能一**：自动创建 AGENTS.md/CLAUDE.md，扫描代码写入 system 文档 |
| 需求散落在聊天记录里，换会话就丢失 | **功能二**：需求自动结构化写入，质量门禁把关，plan/spec/test 完整驱动 |
| Agent 踩过的坑下次还踩 | **功能三**：对话经验自动沉淀到 known-pitfalls，文档定期收敛去重 |

## 快速开始

### 1. 项目初始化

将本目录放入项目的 `.trae/skills/doc-driven/`（或其他 Agent 平台的 Skill 目录），然后说：

```
doc-driven 初始化
```

Agent 会自动：
- 创建 `AGENTS.md` 和 `CLAUDE.md`（如不存在）
- 创建 `docs/` 完整目录结构
- 扫描现有代码，识别模块/数据/接口，写入 `docs/system/`
- 创建需求索引

### 2. 需求驱动执行

直接说需求，Agent 自动完成全流程：

```
我想做一个多端同步功能
```

```
这个需求可以开始做了
```

Agent 会自动：创建需求 → 质量门禁 → 生成执行计划 → 生成执行规格 → 验证证据

### 3. 对话总结与收敛

```
总结对话
```

Agent 自动：提取对话中的经验教训写入 system 文档 → 文档收敛去重归档

## 核心方法论

### 三大功能

| 功能 | 触发词 | 产出 |
| --- | --- | --- |
| 项目初始化 | `doc-driven 初始化` | AGENTS.md、CLAUDE.md、docs/ 目录、system/ 基线文档 |
| 需求驱动执行 | 自然语言描述需求 / `执行` | requirement → plan → spec → test 完整链路 |
| 对话总结与收敛 | `总结对话` / `整理文档` | known-pitfalls 更新、文档去重、归档、重建索引 |

### 写回规则

改完代码 ≠ 完成任务。以下变更必须同步更新文档：

| 变更类型 | 必须更新的文档 |
| --- | --- |
| 架构/模块变化 | `current-architecture.md` |
| 产品规则/交互变化 | `product-rules.md` |
| 字段/状态机变化 | `data-model.md` |
| 接口变化 | `api-inventory.md` |
| 踩坑经验 | `known-pitfalls.md` |
| 重要演进 | `changelog.md` |

### 质量门禁

需求进入开发前必须通过 9 题检查——答不上来标 `待验证`，不猜。

## 目录结构

```
doc-driven/
├── SKILL.md                          # Skill 主文件
├── README.md                         # 本文件
└── templates/
    ├── agents-template.md            # AGENTS.md 模板
    ├── claude-template.md            # CLAUDE.md 模板
    ├── requirement.md                # 需求文档模板
    ├── requirements-index.md         # 需求索引模板
    └── requirement-quality-gate.md   # 质量门禁模板
```

初始化后在项目根目录创建：

```
docs/
├── requirements/          # 需求文档
│   ├── index.md           # 需求索引
│   └── archive/           # 已归档需求
├── superpowers/
│   ├── plans/             # 执行计划
│   ├── specs/             # 执行规格
│   └── tests/             # 验证证据
├── system/                # 系统长期事实
├── design/                # UI 设计
└── promo/                 # 宣传物料
```

## 支持的平台

平台无关设计：Trae IDE、Claude Code、Cursor、GitHub Copilot、Codex、Windsurf。

## License

MIT