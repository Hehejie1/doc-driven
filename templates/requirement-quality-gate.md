# Requirement Quality Gate

在把 `docs/requirements/` 交给 Superpowers plan/spec 前，必须通过此检查。

## 必须通过

- [ ] 保留了用户原始意图或原话摘要。
- [ ] 写清楚用户、场景、痛点和业务价值。
- [ ] 目标描述的是产品结果，不是实现步骤。
- [ ] 明确非目标，避免范围膨胀。
- [ ] 当前系统事实来自代码、文档或用户确认；未知项标记为 `待验证`。
- [ ] 验收标准是可验证的 `GIVEN/WHEN/THEN`、`WHEN/THEN` 或 `SHALL`。
- [ ] 至少包含正常路径、失败路径或边界场景。
- [ ] 写清平台、性能、隐私、安全、兼容或成本约束中相关项。
- [ ] 检查了 `product-rules.md`、`data-model.md`、`api-inventory.md`、`known-pitfalls.md` 的冲突。
- [ ] 开放问题只保留会阻塞 plan/spec 的问题。
- [ ] 没有写入密钥、token、隐私数据或原始用户素材。

## 不允许

- [ ] 把需求文档写成实施方案。
- [ ] 在需求层新增未确认的数据库字段、接口字段或 native 入口。
- [ ] 用"优化、提升、完善、支持"等词但没有可观察验收。
- [ ] 只写 happy path，不写失败或边界情况。
- [ ] 把 UI 视觉风格写在需求层；应进入 `docs/design/`。
- [ ] 把宣传文案或发布素材写在需求层；应进入 `docs/promo/`。
- [ ] 把测试执行日志写在需求层；应进入 `docs/superpowers/tests/`。

## Superpowers Handoff

只有当状态为 `Ready for Plan` 时，才能进入：

```text
docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md
docs/superpowers/specs/YYYY-MM-DD-<feature-name>.md
```

如果计划过程中发现需求冲突或新增产品范围，先回写 requirement，再继续计划。