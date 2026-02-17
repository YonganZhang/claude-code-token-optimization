# 设计模式

## 一、Rule + Skill 联动

Rule 和 Skill 不是二选一，而是分工协作：

- **Rule**（≤10 行）：触发条件 + 一句话摘要 + `→ 先读 skill xxx`
- **Skill**：完整的详细规则和参考内容

**示例：**
```
# environment.md（Rule，5行）
- 项目文档/CLAUDE.md/初始化项目 → 先读 skill `project-doc-guide`
- 创建/修改 Rule 或 Skill → 先读 skill `skill-creator`
```
AI 每轮看到触发条件（5行），只在匹配时才加载 Skill 全文。

**创建 Rule 时：** 始终评估是否需要配套 Skill。如果 Rule 超过 10 行，说明详情应该拆到 Skill。

### Description 触发替代 Rule

Skill description（frontmatter）每轮已常驻上下文（1行）。若 description 写明触发条件 + "必须"，可直接替代 Rule：

| 场景 | 用 Rule | 用 Skill description |
|------|---------|---------------------|
| 每条命令都要遵守（代理、路径） | ✅ | ✗ |
| 特定任务触发的规范（写代码、调试） | ✗ | ✅ 省掉 ≤10 行固定成本 |

## 二、Skill 多文件渐进式披露

单个大 SKILL.md 一加载就全进上下文，无法省 token。解决方案：

```
my-skill/
├── SKILL.md              # 索引（≤50行）：目录 + 各子文件一句话摘要
├── references/
│   ├── detail-a.md       # 详情（AI 按需 Read）
│   └── detail-b.md
├── scripts/              # 可执行脚本
└── assets/               # 输出资源
```

**SKILL.md 是索引，不是文档。** 内容超过 50 行时必须拆分。
SKILL.md 中用 `详见 references/xxx.md` 指引 AI 按需读取。

## 三、项目文档拆分（CLAUDE.md → 多项目 Skill）

CLAUDE.md 每轮全文加载，是仅次于 Rule 的固定成本大户。

**CLAUDE.md 只放项目独有规范：**
- 项目身份（一句话）
- 项目独有的代码/Git 规范（与全局 rules 去重）
- 常用开发命令
- 操作约束
- **禁止放入**全局 rules 已有内容

**查阅内容拆成多个小项目 Skill：**

| Skill 名 | 内容 | 条目格式 |
|-----------|------|---------|
| `project-modules` | 模块目录 | 模块名 → 职责 → 入口 → 关键导出 |
| `project-data` | 数据目录 | 表/配置/结构 → 位置 → schema |
| `project-pipelines` | 构建部署流程 | 流程名 → 配置文件 → 关键步骤 |

按项目复杂度选 2-5 个，小项目可只用 `project-modules`。

**判断标准：**

| 问题 | 结论 |
|------|------|
| 删掉后 AI 行为不变？ | 不放 CLAUDE.md |
| 全局 rules/skills 已覆盖？ | 不放 CLAUDE.md |
| 告诉 AI "哪里有什么可复用"？ | 放 skill 元信息索引 |
| 实现细节且会频繁变化？ | 不存储，按需读源码 |

## 四、索引自维护

通过 environment.md 中的触发器实现：

```
- 任务完成后，若新增/重构了模块或接口 → 批量更新 .claude/skills/project-* 索引
```

**关键设计：** "任务完成后"+"批量"，避免每个小改动都触发读写。

局限性：尽力触发，非 100% 保证。索引过时时可手动说"更新项目 skill 索引"。

## 五、Skill 瘦身 + 持久日志

### 瘦身清单

| 检查项 | 动作 |
|--------|------|
| PreToolUse hook 每次读文件？ | 删除，改为"重大决策前手动 Read" |
| 多个 plan 文件？ | 合并为 1 个 |
| 模板文件？ | 内联到 SKILL.md（≤15行） |
| 与内置工具重叠？（TodoWrite） | 明确分工，删除重叠 |
| 与其他 Skill 重叠？ | 删除，引用已有 Skill |

案例：plan-with-files 从 249行Skill+3hook+10文件 → 精简为 12行Rule。

### 持久日志（跨会话）

通过 Rule `token-optimization-plan.md` 实现，记录标准：**对项目有推进价值的交互**（任务/问题/决策），不记录闲聊和简单查询。

| 工具 | 生命周期 | 内容 |
|------|---------|------|
| TodoWrite | 当前会话 | 实时步骤进度 |
| plan.md | 跨会话 | 任务、问题、决策 |

新会话若 `docs/plan/plan.md` 存在，先读取恢复上下文。
