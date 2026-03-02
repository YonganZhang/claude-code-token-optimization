<p align="center">
  <h1 align="center">🧠 Token Optimization Plan (TOP)</h1>
  <p align="center">
    <strong>省 60% Token，让 AI 只看它需要的信息</strong><br>
    适用于 Claude Code &amp; Codex
  </p>
  <p align="center">
    <a href="#-快速开始">⚡ 快速开始</a> · <a href="#-核心架构">🏗️ 架构</a> · <a href="#-五大设计模式">🎯 模式</a> · <a href="docs/posts/xiaohongshu-series.md">📱 小红书</a>
  </p>
  <p align="center">
    <img src="https://img.shields.io/badge/Claude_Code-支持-blueviolet?style=for-the-badge" alt="Claude Code">
    <img src="https://img.shields.io/badge/Codex-支持-orange?style=for-the-badge" alt="Codex">
    <img src="https://img.shields.io/badge/Token_Saved-60%25+-success?style=for-the-badge" alt="Token Saved 60%+">
    <img src="https://img.shields.io/github/license/YonganZhang/code-token-optiization?style=for-the-badge" alt="License">
  </p>
</p>

---

## ⚡ 快速开始

> 本仓库本身就是一个标准 Skill 包（根目录有 `SKILL.md` + `references/`），可直接安装。

### Claude Code 用户

```bash
# 1. 安装 Skill
git clone https://github.com/YonganZhang/code-token-optiization.git \
  ~/.claude/skills/top

# 2. 创建触发 Rule
cat > ~/.claude/rules/environment.md << 'EOF'
# Environment
- 项目文档/CLAUDE.md/初始化项目 → 先读 skill `project-doc-guide`
- 创建/修改 Rule 或 Skill → 先读 skill `skill-creator`
- 任务完成后新增模块 → 批量更新 .claude/skills/project-*
EOF

# 3. 复制示例 Rules（可选）
cp examples/rules/*.md ~/.claude/rules/
```

👉 详细步骤见 [references/setup-claude-code.md](references/setup-claude-code.md)

### Codex 用户

```bash
# 1. 安装 Skill
git clone https://github.com/YonganZhang/code-token-optiization.git \
  ~/.codex/skills/top

# 2. 在 AGENTS.md 中加入触发索引
cat >> AGENTS.md << 'EOF'

## Skills 使用规则
- 初始化项目文档时 → 先加载 skill `project-doc-guide`
- 创建新 Skill 时 → 先加载 skill `skill-creator`
- 任务完成后新增模块 → 批量更新 project-* 索引
EOF
```

👉 详细步骤见 [references/setup-codex.md](references/setup-codex.md)

### 验证安装

新对话中说「帮我初始化项目文档」，观察 AI 是否：
- ✅ 加载了 `project-doc-guide` skill
- ✅ 生成了精简的项目说明（CLAUDE.md / AGENTS.md）
- ✅ 创建了多个项目 Skill（project-modules 等）

---

## 💡 这是什么？

一套通过**分层管理**实现 AI 编程助手 Token 消耗降低 50-60% 的完整方法论。

### 📊 Before vs After

| | 优化前 | 优化后 |
|---|---|---|
| 项目说明 | 200 行 × 每轮 | **60 行** × 每轮 |
| 规则文件 | 50+ 行 × 每轮 | **46 行** × 每轮 |
| 查阅信息 | 全塞项目说明 | **按需加载** Skill |
| 工具 schema | 全部预加载 | **延迟加载** |
| **20 轮对话总加载** | **~5000 行** | **~2100 行** |
| **节省** | — | **🎯 55-65%** |

### 两个平台，同一套方法

| 概念 | Claude Code | Codex |
|------|------------|----------|
| 项目说明 | CLAUDE.md | AGENTS.md |
| 规则文件 | ~/.claude/rules/*.md | ~/.codex/rules/*.md |
| Skills 位置 | ~/.claude/skills/ | ~/.codex/skills/ |
| Skill 结构 | SKILL.md + references/ | SKILL.md + references/ |

---

## 🏗️ 核心架构

### 四层渐进式披露

```
          ┌─────────────────────┐
    L0    │  规则文件（≤10行）    │  ← 每轮加载，触发索引
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

### 1️⃣ 触发索引 + Skill 联动

> 规则文件不存详情，只做触发器。

```
触发流程：

用户说"帮我初始化项目"
        │
        ▼
规则文件匹配到"初始化项目"（5行，每轮已加载）
        │
        ▼
加载 project-doc-guide Skill（53行，仅此时加载）
        │
        ▼
AI 按规则生成项目说明 + 多个项目 Skill
```

**对比：** 不联动 = 53行 × 每轮。联动后 = 5行 × 每轮 + 53行 × 偶尔。

### 2️⃣ Skill 多文件渐进式披露

```
my-skill/
├── SKILL.md              # 📋 索引（≤50行）
├── references/
│   ├── detail-a.md       # 📖 详情（AI 按需 Read）
│   └── detail-b.md
├── scripts/              # ⚙️ 可执行脚本
└── assets/               # 📁 输出资源
```

SKILL.md 是目录，不是文档。超过 50 行必须拆分。

### 3️⃣ 项目说明瘦身

| 留在项目说明 ✅ | 拆到项目 Skill 📦 |
|---|---|
| 项目身份（一句话） | `project-modules` 模块索引 |
| 项目独有代码规范 | `project-data` 数据/配置索引 |
| 常用开发命令 | `project-pipelines` 构建流程 |

**判断标准：** 删掉后 AI 还能正确写代码？→ 拆出去。

### 4️⃣ 索引自维护

```
规则文件中的触发器：
- 任务完成后，若新增/重构了模块或接口 → 批量更新 project-* 索引
```

### 5️⃣ 工具延迟加载

Claude Code: `"ENABLE_TOOL_SEARCH": "true"`

---

## 📁 仓库结构

```
.
├── SKILL.md                            # 🌟 Skill 包入口（安装后 AI 可直接使用）
├── references/                         # 详细参考文档
│   ├── cost-model.md                   # 上下文成本模型
│   ├── patterns.md                     # 五大设计模式详解
│   ├── setup-claude-code.md            # Claude Code 搭建指南
│   └── setup-codex.md                  # Codex 搭建指南
├── examples/                           # 示例配置文件
│   ├── rules/                          # 示例 Rules（每个 ≤10 行）
│   └── skills/                         # 示例 Skills
│       ├── token-optimization/         # 本体系 Skill（多文件示范）
│       └── project-doc-guide/          # 项目文档拆分规则
└── docs/posts/                         # 小红书系列帖文案
```

---

## 🤔 FAQ

**Q: 这套方案适用于哪些工具？**
A: Claude Code（VSCode 扩展 / CLI）和 Codex。核心概念适用于任何使用 Skills 的 AI 编程助手。

**Q: 小项目也需要吗？**
A: 小项目可以只用 `project-modules` 一个 Skill。核心收益来自项目说明瘦身和规则精简。

**Q: AI 真的会按触发器去读 Skill 吗？**
A: 大概率会。触发词设计覆盖了常见表述。如果没触发，在指令中加一句「先读 skill xxx」。

**Q: 索引过时了怎么办？**
A: 直接说「更新项目 skill 索引」，AI 会重新扫描并更新。

---

## 📜 License

MIT — 随意使用、修改、分享。

---

<p align="center">
  <strong>⭐ 如果这套方案帮到了你，给个 Star 吧！</strong>
</p>
