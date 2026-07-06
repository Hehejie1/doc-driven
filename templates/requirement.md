# <需求标题>

日期：YYYY-MM-DD
作者 / Agent：
状态：Draft | Clarifying | Ready for Plan | Planned | Implemented | Superseded
关联对话 / 任务：
关联系统文档：

## 1. 原始需求

保留用户原始意图摘要。必要时引用关键原话。

## 2. 背景和问题

- 当前用户是谁：
- 当前场景是什么：
- 当前痛点是什么：
- 为什么现在要做：

## 3. 目标结果

用产品语言描述用户最终要得到什么结果，不写实现步骤。

- 目标 1：
- 目标 2：
- 目标 3：

## 4. 非目标

明确本次不做什么，避免范围膨胀。

- 不做：
- 不做：
- 不做：

## 5. 当前系统事实

只写已经从代码、文档或用户确认中得到的事实。未知内容写 `待验证`。

- 已有类似能力：
- 当前入口 / 页面 / API：
- 当前数据来源和写入位置：
- 当前限制：
- 相关 known pitfalls：

## 6. 用户故事

- 作为 <用户类型>，我希望 <完成动作 / 获得能力>，以便 <业务价值 / 用户价值>。

## 7. 验收标准

使用可验证表达。优先 `GIVEN / WHEN / THEN` 或 `WHEN / THEN`。

1. GIVEN <前置状态>，WHEN <用户动作或系统事件>，THEN <可观察结果>。
2. WHEN <条件>，THEN <可观察结果>。
3. The system SHALL <行为> when <条件>。

## 8. 输入 / 输出样例

### 输入

- 

### 输出

- 

### 反例或失败样例

- 

## 9. 约束

- 平台 / 设备：
- 性能 / 耗时：
- 隐私 / 安全：
- 兼容性：
- 成本：
- 时间：
- 第三方依赖 / License：

## 10. 边界场景

- 空数据：
- 权限拒绝：
- 网络失败：
- 任务中断：
- 旧数据兼容：
- 大数据量：
- 其他：

## 11. 开放问题

只有阻塞 plan/spec 的问题才放这里。

1. 
2. 
3. 

## 12. 与系统事实的冲突检查

- 是否和 `product-rules.md` 冲突：
- 是否和 `data-model.md` 字段语义冲突：
- 是否和 `api-inventory.md` 接口语义冲突：
- 是否会重新引入 `known-pitfalls.md` 中的问题：
- 是否影响本地数据或线上服务：
- 是否影响用户已确认交互：

## 13. Superpowers 交接

- 是否已达到 Ready for Plan：
- 推荐下一步：
  - [ ] 使用 Superpowers 生成 `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`
  - [ ] 使用 Superpowers 生成 `docs/superpowers/specs/YYYY-MM-DD-<feature>.md`
  - [ ] 继续向用户澄清
- 计划必须覆盖的验收标准：
- 计划必须补充的验证：