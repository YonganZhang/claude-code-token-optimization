# 从零搭建指南

## 搭建步骤

### 1. settings.json 基础配置

```json
{
  "env": {
    "ENABLE_TOOL_SEARCH": "true"
  },
  "permissions": {
    "allow": ["Bash(git add:*)", "Bash(git commit:*)"]
  }
}
```

### 2. 创建全局 Rules（精简触发器）

每个 Rule ≤ 10 行，只放必须每轮遵守的约束：

- 路径/Shell 规范
- 工具调用约束（写入行数限制）
- 网络请求约束（代理、搜索工具）
- 环境标识 + Skill 触发索引（environment.md）

### 3. 创建全局 Skills

按需加载的详细内容，每个 SKILL.md ≤ 50 行索引，超出部分放 references/：

- `token-optimization` — 本体系文档
- `project-doc-guide` — 项目文档拆分规则
- `skill-creator` — 创建新 Skill 的向导
- 其他按需：MCP 工具目录、调试方法论、存档回滚等

### 4. 配置 MCP + Tool Search

在 settings.json 设置 `ENABLE_TOOL_SEARCH: "true"`，配合 `mcp-directory` skill 提供搜索关键词。

### 5. 配置 Hooks（可选）

- SessionStart → git init 脚本
- Stop → 会话结束自动存档

### 6. 项目级配置

进入具体项目后：
1. 创建精简 CLAUDE.md（≤60 行，只含项目独有规范）
2. 创建项目 Skills（project-modules、project-data 等元信息索引）
3. 详见 skill `project-doc-guide`

## 当前文件结构参考

```
~/.claude/
├── settings.json              # 全局设置（env、permissions、hooks）
├── rules/
│   ├── chunked-writing.md     # 写入行数限制
│   ├── windows-bash.md        # Unix路径规范
│   ├── proxy-china.md         # 代理设置
│   ├── web-search.md          # 搜索工具策略
│   ├── token-optimization-plan.md # 分层策略 + 持久日志
│   └── environment.md         # IDE标识 + Skill触发索引
├── skills/
│   ├── token-optimization/    # 本体系（多文件结构示范）
│   ├── project-doc-guide/     # 项目文档拆分规则
│   ├── skill-creator/         # 创建新Skill向导
│   ├── mcp-directory/         # MCP工具搜索关键词
│   ├── checkpoint/            # 存档与回滚
│   ├── python/                # Python环境规范
│   ├── config-paths/          # 配置文件路径
│   ├── systematic-debugging/  # 调试方法论
│   └── drawio-diagrams/       # 图表生成
└── scripts/
    ├── auto-checkpoint.js     # Stop hook
    └── session-git-init.js    # SessionStart hook
```

## 新机器迁移

按顺序复制：settings.json → rules/ → skills/ → scripts/，按目标环境调整路径规范和代理设置。
