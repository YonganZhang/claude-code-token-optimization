# Claude Code 从零搭建指南

> 同时参考 OpenClaw 版本：`references/setup-openclaw.md`

## Step 1：开启 Tool Search

在 `~/.claude/settings.json` 中加入：

```json
{
  "env": {
    "ENABLE_TOOL_SEARCH": "true"
  }
}
```

效果：所有 MCP 工具 100% 延迟加载，AI 通过 ToolSearch 按需搜索。

## Step 2：安装 Skill

将本仓库复制到 Claude Code 的 skills 目录：

```bash
git clone https://github.com/YonganZhang/claude-code-token-optimization.git \
  ~/.claude/skills/token-optimization
```

或者只复制核心文件：
```bash
mkdir -p ~/.claude/skills/token-optimization/references
cp SKILL.md ~/.claude/skills/token-optimization/
cp references/*.md ~/.claude/skills/token-optimization/references/
```

## Step 3：创建触发 Rule

```bash
cat > ~/.claude/rules/environment.md << 'EOF'
# Environment
- IDE: VSCode with Claude extension
- 项目文档/CLAUDE.md/初始化项目 → 先读 skill `project-doc-guide`
- 创建/修改 Rule 或 Skill → 先读 skill `skill-creator`
- 任务完成后，若新增/重构了模块或接口 → 批量更新 .claude/skills/project-*
EOF
```

这 5 行 Rule 是整套体系的入口。AI 每轮看到触发条件，匹配时才加载对应 Skill。

## Step 4：精简你的 CLAUDE.md

CLAUDE.md 每轮全文加载，只保留：

- 项目身份（一句话）
- 项目独有的代码/Git 规范（与全局 Rules 去重）
- 常用开发命令
- 操作约束

**禁止放入**全局 Rules 已有内容（代理、路径规范、写入限制等）。

## Step 5：创建项目 Skills

```bash
mkdir -p .claude/skills/project-modules .claude/skills/project-data
```

每个 Skill 的 SKILL.md ≤50 行，表格式元信息索引：

```markdown
---
description: "项目模块索引 — 各模块职责、入口、关键导出"
---
| 模块 | 职责 | 入口 |
|------|------|------|
| auth | 用户认证 | src/auth/index.ts |
| api  | REST API | src/api/router.ts |
```

## Step 6：验证

新对话中说「帮我初始化项目文档」，观察 AI 是否：
- ✅ 读取了 `project-doc-guide` skill
- ✅ 生成了精简的 CLAUDE.md（≤60 行）
- ✅ 创建了多个项目 Skill（project-modules 等）

## 可选：更多示例 Rules

本仓库 `examples/rules/` 下有 6 个示例 Rule，可按需复制：

```bash
cp examples/rules/*.md ~/.claude/rules/
```

## Claude Code vs OpenClaw 对照

| 概念 | Claude Code | OpenClaw |
|------|------------|----------|
| 项目说明 | CLAUDE.md | SOUL.md |
| 规则文件 | ~/.claude/rules/*.md | TOOLS.md + AGENTS.md |
| Skills 位置 | ~/.claude/skills/ | ~/.openclaw/skills/ |
| Skill 结构 | SKILL.md + references/ | SKILL.md + references/ |
| 延迟加载 | ENABLE_TOOL_SEARCH | 默认已启用 |
