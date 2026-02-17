<p align="center">
  <h1 align="center">🧠 Claude Code Token 优化体系</h1>
  <p align="center">
    <strong>省 60% Token，让 AI 只看它需要的信息</strong>
  </p>
  <p align="center">
    <a href="#快速开始">快速开始</a> · <a href="#核心架构">核心架构</a> · <a href="#五大设计模式">设计模式</a> · <a href="docs/posts/xiaohongshu-series.md">小红书系列</a>
  </p>
  <p align="center">
    <img src="https://img.shields.io/badge/Claude_Code-Token_Optimization-blueviolet?style=for-the-badge" alt="Claude Code Token Optimization">
    <img src="https://img.shields.io/badge/Token_Saved-60%25+-success?style=for-the-badge" alt="Token Saved 60%+">
    <img src="https://img.shields.io/github/license/YonganZhang/claude-code-token-optimization?style=for-the-badge" alt="License">
  </p>
</p>

---

## 💡 这是什么？

一套通过 **Rule / Skill / MCP 分层管理**，实现 Claude Code Token 消耗降低 50-60% 的完整方法论。

> 📦 本仓库自身就是这套方法论的活示范 —— Skill 索引 + references 渐进式披露。直接复制 `examples/` 到你的 `~/.claude/` 即可使用。

## 🔥 为什么需要这个？

Claude Code 每一轮对话，以下内容**全文重复加载**：

```
┌─────────────────────────────────────────────────────┐
│  每轮对话的上下文窗口                                  │
│                                                       │
│  ┌──────────────┐  ← 系统提示词（不可控）              │
│  ├──────────────┤                                     │
│  │ Rules 全文    │  ← ⚠️ 你写了 50 行？每轮 50 行     │
│  ├──────────────┤                                     │
│  │ CLAUDE.md 全文│  ← ⚠️ 你写了 200 行？每轮 200 行   │
│  ├──────────────┤                                     │
│  │ Skill 描述    │  ← ✅ 每个只有 1 行                 │
│  ├──────────────┤                                     │
│  │ MCP 工具      │  ← ⚠️ 默认全部加载                  │
│  ├──────────────┤                                     │
│  │ 对话历史      │                                     │
│  └──────────────┘                                     │
└─────────────────────────────────────────────────────┘
```

### 📊 Before vs After

| | 优化前 | 优化后 |
|---|---|---|
| CLAUDE.md | 200 行 × 每轮 | **60 行** × 每轮 |
| Rules | 50+ 行 × 每轮 | **46 行** × 每轮 |
| 查阅信息 | 全塞 CLAUDE.md | **按需加载** Skill |
| MCP 工具 | 全部预加载 | **Tool Search 延迟加载** |
| **20 轮对话总加载** | **~5000 行** | **~2100 行 + 按需 ~200 行** |
| **节省** | — | **🎯 约 55-65%** |

---

## 🏗️ 核心架构

### 四层渐进式披露

```
          ┌─────────────────────┐
    L0    │   Rule（≤10行）      │  ← 每轮加载，触发索引
          ├─────────────────────┤
    L1    │ Skill SKILL.md      │  ← 触发时加载，目录索引
          │    （≤50行）         │
          ├─────────────────────┤
    L2    │ Skill references/   │  ← AI 按需 Read
          │   *.md 详情文件      │
          ├─────────────────────┤
    L3    │     源码文件         │  ← AI 按需 Read
          └─────────────────────┘
               越往下越大，加载越少
```

**核心思想：每一层只在需要时才加载下一层。**

---

## 🎯 五大设计模式

### 1️⃣ Rule + Skill 联动

> Rule 不存详情，只做触发器。

```markdown
# ~/.claude/rules/environment.md（5行，每轮加载）

- IDE: VSCode with Claude extension
- 项目文档/CLAUDE.md/初始化项目 → 先读 skill `project-doc-guide`
- 创建/修改 Rule 或 Skill → 先读 skill `skill-creator`
- 任务完成后新增模块 → 批量更新项目索引
```

```
触发流程：

用户说"帮我初始化项目"
        │
        ▼
environment.md 匹配到"初始化项目"（5行，每轮已加载）
        │
        ▼
加载 project-doc-guide Skill（53行，仅此时加载）
        │
        ▼
AI 按规则生成 CLAUDE.md + 多个项目 Skill
```

**对比：** 不联动 = 53行 × 每轮。联动后 = 5行 × 每轮 + 53行 × 偶尔。

### 2️⃣ Skill 多文件渐进式披露

> SKILL.md 是索引，不是文档。

```
my-skill/
├── SKILL.md              # 📋 索引（≤50行）：目录 + 摘要
├── references/
│   ├── detail-a.md       # 📖 详情 A（AI 按需 Read）
│   └── detail-b.md       # 📖 详情 B（AI 按需 Read）
├── scripts/              # ⚙️ 可执行脚本
└── assets/               # 📁 输出资源
```

超过 50 行？必须拆分。SKILL.md 中用 `详见 references/xxx.md` 指引 AI 按需读取。

### 3️⃣ CLAUDE.md 瘦身

> 只放项目独有规范，查阅内容拆成多个小项目 Skill。

| 留在 CLAUDE.md ✅ | 拆到项目 Skill 📦 |
|---|---|
| 项目身份（一句话） | `project-modules` 模块索引 |
| 项目独有代码规范 | `project-data` 数据/配置索引 |
| 常用开发命令 | `project-pipelines` 构建流程 |
| 操作约束 | |

**判断标准：** 删掉后 AI 还能正确写代码？→ 拆出去。

### 4️⃣ 索引自维护

```markdown
# environment.md 中的触发器
- 任务完成后，若新增/重构了模块或接口 → 批量更新 .claude/skills/project-* 索引
```

AI 写完新模块后，自动在 `project-modules` 索引里加一行。

### 5️⃣ MCP Tool Search 延迟加载

```json
{ "env": { "ENABLE_TOOL_SEARCH": "true" } }
```

所有 MCP 工具 100% 延迟加载，AI 通过 ToolSearch 按需搜索和加载。

---

## 🚀 快速开始

### Step 1：开启 Tool Search

在 `~/.claude/settings.json` 的 `env` 中加入：

```json
"ENABLE_TOOL_SEARCH": "true"
```

### Step 2：复制示例文件

```bash
# 复制 Rules
cp examples/rules/*.md ~/.claude/rules/

# 复制 Skills
cp -r examples/skills/* ~/.claude/skills/
```

### Step 3：精简你的 CLAUDE.md

对照 `examples/skills/project-doc-guide/SKILL.md` 中的规则，把查阅型内容拆到项目 Skill。

### Step 4：验证

新对话中说「帮我初始化项目文档」，观察 AI 是否：
- ✅ 读取了 `project-doc-guide` skill
- ✅ 生成了精简的 CLAUDE.md
- ✅ 创建了多个项目 Skill（project-modules 等）

---

## 📁 仓库结构

```
examples/
├── rules/                              # 示例 Rules（每个 ≤10 行）
│   ├── environment.md                  # Skill 触发索引
│   ├── chunked-writing.md              # 写入行数限制
│   ├── proxy-china.md                  # 代理设置
│   ├── web-search.md                   # 搜索工具策略
│   ├── windows-bash.md                 # Windows 路径规范
│   └── token-optimization-plan.md      # 分层策略 + 持久日志
├── skills/
│   ├── token-optimization/             # 🌟 本体系 Skill（多文件示范）
│   │   ├── SKILL.md                    # 索引（36行）
│   │   └── references/
│   │       ├── cost-model.md           # 上下文成本模型
│   │       ├── patterns.md             # 五大设计模式详解
│   │       └── setup.md               # 从零搭建指南
│   └── project-doc-guide/             # 项目文档拆分规则
│       └── SKILL.md
docs/
└── posts/
    └── xiaohongshu-series.md           # 📱 小红书系列帖文案
```

---

## 🤔 FAQ

**Q: 这套方案适用于哪些工具？**
A: Claude Code（VSCode 扩展 / CLI）、任何使用 CLAUDE.md + Skills 的 AI 编程助手。

**Q: 小项目也需要吗？**
A: 小项目可以只用 `project-modules` 一个 Skill。核心收益来自 CLAUDE.md 瘦身和 Rule 精简。

**Q: AI 真的会按触发器去读 Skill 吗？**
A: 大概率会。environment.md 的触发词设计覆盖了常见表述（"初始化项目"、"创建 CLAUDE.md"、"项目文档"）。如果没触发，可以在指令中加一句「先读 skill xxx」。

**Q: 索引过时了怎么办？**
A: 直接说「更新项目 skill 索引」，AI 会重新扫描并更新。

---

## 📜 License

MIT — 随意使用、修改、分享。

---

<p align="center">
  <strong>⭐ 如果这套方案帮到了你，给个 Star 吧！</strong>
</p>
