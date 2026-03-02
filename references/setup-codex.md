# Codex 从零搭建指南

> 同时参考 Claude Code 版本：`references/setup-claude-code.md`

## Step 1：安装 Skill

将本仓库复制到 Codex 的 skills 目录：

```bash
git clone https://github.com/YonganZhang/code-token-optiization.git \
  ~/.codex/skills/top
```

或者只复制核心文件：
```bash
mkdir -p ~/.codex/skills/top/references
cp SKILL.md ~/.codex/skills/top/
cp references/*.md ~/.codex/skills/top/references/
```

## Step 2：配置 AGENTS.md

在 `~/.codex/AGENTS.md` 中加入触发索引：

```markdown
## Skills 使用规则
- 初始化项目文档时 → 先加载 skill `project-doc-guide`
- 创建新 Skill 时 → 先加载 skill `skill-creator`
- 任务完成后新增模块 → 批量更新 project-* 索引
```

## Step 3：创建 Rules

在 `~/.codex/rules/` 下创建规则文件（每个 ≤10 行）：

```bash
cp examples/rules/*.md ~/.codex/rules/
```

## Step 4：创建项目 Skills

```bash
mkdir -p skills/project-modules skills/project-data
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

## Step 5：验证

对 agent 说「帮我初始化项目文档」，观察是否：
- ✅ 加载了 `project-doc-guide` skill
- ✅ 生成了精简的项目说明
- ✅ 创建了多个项目 Skill

## Codex vs Claude Code 对照

| 概念 | Claude Code | Codex |
|------|------------|----------|
| 项目说明 | CLAUDE.md | AGENTS.md |
| 规则文件 | ~/.claude/rules/*.md | ~/.codex/rules/*.md |
| Skills 位置 | ~/.claude/skills/ | ~/.codex/skills/ |
| Skill 结构 | SKILL.md + references/ | SKILL.md + references/ |
| Hooks | 原生支持 | 不支持（AGENTS.md 替代） |
