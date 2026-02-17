---
description: "项目文档生成指南 — 创建/修改 CLAUDE.md 时的拆分规则"
---

# 项目文档生成指南

## CLAUDE.md 原则

只放项目独有规范，与全局 rules/skills 去重：

- 项目身份（一句话）
- 项目独有的代码/Git 规范
- 常用开发命令
- 操作约束

**禁止放入**全局 rules 已有内容（代理、路径规范、写入限制等）。

## 项目 Skills = 元信息索引

查阅内容拆成多个小 skill（单个大 skill 一加载就全进来，无法省 token）。

每个 skill ≤ 50 行，只存稳定结构信息：

| Skill 名 | 内容 | 条目格式 |
|-----------|------|---------|
| `project-modules` | 模块目录 | 模块名 → 职责 → 入口 → 关键导出 |
| `project-data` | 数据目录 | 表/配置/结构 → 位置 → schema |
| `project-pipelines` | 构建部署流程 | 流程名 → 配置文件 → 关键步骤 |

按项目复杂度选 2-5 个。小项目可只用 `project-modules`。

description 写清领域关键词，让 AI 按需加载。

## 判断标准

| 问题 | 结论 |
|------|------|
| 删掉后 AI 行为不变？ | 不放 CLAUDE.md |
| 全局 rules/skills 已覆盖？ | 不放 CLAUDE.md |
| 告诉 AI "哪里有什么可复用"？ | 放 skill 元信息索引 |
| 实现细节且会频繁变化？ | 不存储，按需读源码 |

## 生成流程

1. `CLAUDE.md` — 项目独有规范（先与全局 rules 去重）
2. `.claude/skills/project-modules/SKILL.md` — 模块索引（必建）
3. `.claude/skills/project-data/SKILL.md` — 数据索引（有数据库/配置时建）
4. `.claude/skills/project-pipelines/SKILL.md` — 流程索引（有 CI/构建时建）

## 已有 CLAUDE.md 的拆分

1. 逐行检查是否与全局 rules 重复 → 删除
2. 规范约束 → 留 CLAUDE.md；查阅信息 → 按领域分到对应 skill
