AppliesTo: both
# Token Optimization Plan (TOP)

## 分层策略
- 常驻规则 → Rule（≤10行），特定任务 → Skill 按需加载
- 闲聊/简单查询 → 不记录项目日志

## Debug
- 同一问题连续 2 次未解决 → 最大循环 5 次后停止汇报

## Plan
- `Yongan/docx/00-总索引.md` 由 Hook 自动注入，视为权威上下文
- 复杂任务（≥3 步）且无索引时，建议初始化 → 加载 skill `top`
- 完成动作后更新索引状态和 progress.md → 详细规则见 skill `top`
