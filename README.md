# Claude Code Token 优化体系

一套通过 Rule / Skill / MCP 分层管理，实现 Claude Code **Token 消耗降低 50-60%** 的完整方法论。

> 本仓库自身就是这套方法论的活示范 —— Skill 索引 + references 渐进式披露。

## 为什么需要这个？

Claude Code 的上下文窗口是有限的。每一轮对话，以下内容**全文加载**：

| 类型 | 加载方式 | 你能控制吗？ |
|------|---------|------------|
| 系统提示词 | 每轮全文 | ❌ |
| Rules | 每轮全文 | ✅ 精简到 ≤10 行 |
| CLAUDE.md | 每轮全文 | ✅ 拆分到 Skill |
| Skill descriptions | 每轮 1 行 | ✅ 写好触发词 |
| MCP 工具 schema | 每轮全部（默认） | ✅ 开启 Tool Search |

**不优化的代价：** 200 行 CLAUDE.md × 20 轮对话 = 4000 行重复加载。
**优化后：** 60 行 CLAUDE.md × 20 轮 + 按需加载 = 1200+200 行。省了 65%。

## 核心架构：四层渐进式披露

```
L0  Rule（≤10行）         ← 每轮加载，触发索引
L1  Skill SKILL.md（≤50行） ← 触发时加载，目录索引
L2  Skill references/*.md    ← AI 按需 Read，详细内容
L3  源码                     ← AI 按需 Read，实现细节
```

每一层只在需要时才加载下一层，避免一次性把所有信息塞进上下文。

## 五大设计模式

### 1. Rule + Skill 联动

Rule 不存详情，只做触发器：

```markdown
# environment.md（Rule，5行）
- 项目文档/CLAUDE.md/初始化项目 → 先读 skill `project-doc-guide`
- 创建/修改 Rule 或 Skill → 先读 skill `skill-creator`
```

AI 每轮看到 5 行触发条件，只在匹配时才加载 Skill 全文。

### 2. Skill 多文件渐进式披露

```
my-skill/
├── SKILL.md              # 索引（≤50行）
├── references/
│   ├── detail-a.md       # 详情（AI 按需 Read）
│   └── detail-b.md
```

SKILL.md 是目录，不是文档。超过 50 行必须拆分。

### 3. CLAUDE.md 瘦身

CLAUDE.md 只放**项目独有规范**，查阅内容拆成多个小项目 Skill：

| Skill | 内容 |
|-------|------|
| `project-modules` | 模块名 → 职责 → 入口 → 关键导出 |
| `project-data` | 表/配置 → 位置 → schema |
| `project-pipelines` | 流程 → 配置文件 → 关键步骤 |

### 4. 索引自维护

```markdown
# environment.md 中的触发器
- 任务完成后，若新增/重构了模块或接口 → 批量更新 .claude/skills/project-* 索引
```

### 5. MCP Tool Search 延迟加载

```json
{ "env": { "ENABLE_TOOL_SEARCH": "true" } }
```

所有 MCP 工具 100% 延迟加载，AI 通过 ToolSearch 按需搜索。

## 快速开始

### 1. 开启 Tool Search

在 `~/.claude/settings.json` 的 `env` 中加入 `"ENABLE_TOOL_SEARCH": "true"`。

### 2. 创建触发 Rule

```markdown
# ~/.claude/rules/environment.md
- IDE: VSCode with Claude extension
- 项目文档/CLAUDE.md/初始化项目 → 先读 skill `project-doc-guide`
- 创建/修改 Rule 或 Skill → 先读 skill `skill-creator`
- 任务完成后，若新增/重构了模块或接口 → 批量更新 .claude/skills/project-*
```

### 3. 复制示例 Skills

将 `examples/skills/` 下的文件复制到 `~/.claude/skills/`。

### 4. 精简你的 CLAUDE.md

对照 `examples/` 中的示例，把查阅型内容拆到项目 Skill。

## 文件结构

```
examples/
├── rules/                          # 示例 Rules（每个 ≤10 行）
│   ├── environment.md
│   ├── chunked-writing.md
│   └── ...
├── skills/
│   └── token-optimization/         # 本体系 Skill（多文件示范）
│       ├── SKILL.md                # 索引（36行）
│       └── references/
│           ├── cost-model.md       # 成本模型
│           ├── patterns.md         # 设计模式
│           └── setup.md            # 搭建指南
docs/
└── posts/                          # 小红书系列帖文案
```

## 适用场景

- Claude Code（VSCode 扩展 / CLI）
- 任何使用 CLAUDE.md + Skills 的 AI 编程助手
- 项目规模：中大型（10+ 模块）效果最明显

## License

MIT
