# OpenClaw 从零搭建指南

> 同时参考 Claude Code 版本：`references/setup-claude-code.md`

## Step 1：安装 Skill

将本仓库复制到 OpenClaw 的 managed skills 目录：

```bash
git clone https://github.com/YonganZhang/claude-code-token-optimization.git \
  ~/.openclaw/skills/token-optimization
```

或者只复制核心文件：
```bash
mkdir -p ~/.openclaw/skills/token-optimization/references
cp SKILL.md ~/.openclaw/skills/token-optimization/
cp references/*.md ~/.openclaw/skills/token-optimization/references/
```

## Step 2：配置 AGENTS.md

在你的 agent workspace 的 `AGENTS.md` 中加入触发索引：

```markdown
## Skills 使用规则
- 初始化项目文档时 → 先加载 skill `project-doc-guide`
- 创建新 Skill 时 → 先加载 skill `skill-creator`
- 任务完成后新增模块 → 批量更新 project-* 索引
```

## Step 3：瘦身 SOUL.md

SOUL.md 每轮全文加载，只放：

- Agent 个性和核心行为
- 项目独有规范
- **不放**查阅型内容（拆到项目 Skill）

## Step 4：精简 TOOLS.md

TOOLS.md 只放环境信息（凭据、路径、SSH），不放领域知识。

领域知识应该放在 Skills 的 references/ 里。

## Step 5：创建项目 Skills

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

## Step 6：验证

对 agent 说「帮我初始化项目文档」，观察是否：
- ✅ 加载了 `project-doc-guide` skill
- ✅ 生成了精简的 SOUL.md
- ✅ 创建了多个项目 Skill

## OpenClaw vs Claude Code 对照

| 概念 | Claude Code | OpenClaw |
|------|------------|----------|
| 项目说明 | CLAUDE.md | SOUL.md |
| 规则文件 | ~/.claude/rules/*.md | TOOLS.md + AGENTS.md |
| Skills 位置 | ~/.claude/skills/ | ~/.openclaw/skills/ |
| Skill 结构 | SKILL.md + references/ | SKILL.md + references/ |
| 延迟加载 | ENABLE_TOOL_SEARCH | 默认已启用 |
| 路径引用 | 相对路径 | {baseDir}/references/ |
