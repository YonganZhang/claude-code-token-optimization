# 设计模式

以下模式同时适用于 Claude Code 和 OpenClaw。

## 一、触发索引 + Skill 联动

规则文件不存详情，只做触发器：

**Claude Code 示例（environment.md Rule，5行）：**
```
- 项目文档/初始化项目 → 先读 skill `project-doc-guide`
- 创建/修改 Rule 或 Skill → 先读 skill `skill-creator`
```

**OpenClaw 示例（AGENTS.md，加几行）：**
```
## Skills 使用规则
- 初始化项目文档时 → 先加载 skill `project-doc-guide`
- 创建新 Skill 时 → 先加载 skill `skill-creator`
```

**原则：** 规则文件超过 10 行，说明详情应拆到 Skill。

## 二、Skill 多文件渐进式披露

单个大 SKILL.md 一加载就全进上下文，无法省 token。

```
my-skill/
├── SKILL.md              # 索引（≤50行）
├── references/
│   ├── detail-a.md       # 详情（AI 按需 Read）
│   └── detail-b.md
├── scripts/              # 可执行脚本
└── assets/               # 输出资源
```

**SKILL.md 是索引，不是文档。** 超过 50 行必须拆分。

OpenClaw 用 `{baseDir}/references/xxx.md` 引用，Claude Code 用相对路径。

## 三、项目说明瘦身

| | Claude Code | OpenClaw |
|---|---|---|
| 项目说明文件 | CLAUDE.md | SOUL.md |
| 原则 | 只放项目独有规范 | 只放个性和核心行为 |

查阅内容拆成多个小项目 Skill：

| Skill 名 | 内容 |
|-----------|------|
| `project-modules` | 模块名 → 职责 → 入口 → 关键导出 |
| `project-data` | 表/配置 → 位置 → schema |
| `project-pipelines` | 流程 → 配置文件 → 关键步骤 |

**判断标准：** 删掉后 AI 还能正确写代码？能 → 拆出去。

## 四、索引自维护

通过规则文件中的触发器：

```
- 任务完成后，若新增/重构了模块或接口 → 批量更新 project-* 索引
```

"任务完成后"+"批量"，避免每个小改动都触发。

## 五、Skill 瘦身

| 检查项 | 动作 |
|--------|------|
| Hook 每次读文件？ | 删除，改为手动 Read |
| 多个 plan 文件？ | 合并为 1 个 |
| 模板文件？ | 内联到 SKILL.md（≤15行） |
| 与其他 Skill 重叠？ | 删除，引用已有 Skill |
