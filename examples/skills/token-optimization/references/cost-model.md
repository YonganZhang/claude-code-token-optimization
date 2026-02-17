# 上下文成本模型

## 成本分类

| 类型 | 固定成本（每轮） | 按需成本 |
|------|-----------------|---------|
| Rule | 全文 | 无 |
| Skill | 一行 description | 读取时加载全文 |
| MCP（无 Tool Search） | 所有工具 schema | 调用返回结果 |
| MCP（有 Tool Search） | ToolSearch 描述 + 工具名列表 | 搜索+调用结果 |
| CLAUDE.md | 全文 | 无 |
| Hook stdout | SessionStart stdout | 无 |

**关键洞察：** Rule 和 CLAUDE.md 是最贵的，因为每轮都全文加载。Skill 最便宜，只有 description 一行是固定成本。

## MCP 优化：Tool Search 延迟加载

在 `~/.claude/settings.json` 的 `env` 中设置：

```json
"ENABLE_TOOL_SEARCH": "true"
```

| 值 | 行为 |
|----|------|
| `auto` | MCP 工具超过上下文 10% 时启用（默认） |
| `true` | 始终启用（推荐） |
| `false` | 禁用，全部预加载 |

配合 `mcp-directory` skill 提供工具搜索关键词索引。

### 按项目 MCP 配置

| 作用域 | 配置位置 | 效果 |
|--------|---------|------|
| Local | `~/.claude.json` → `projects.<路径>` | 只在该项目 |
| Project | 项目根目录 `.mcp.json` | 可提交 git |
| User | `~/.claude.json` 顶层 `mcpServers` | 所有项目 |

## Rule 精简技巧

Rule 每轮全文加载，必须尽量短：

- 去掉教学性内容，只保留指令
- 去掉代码块格式，用单行内联代码
- 去掉分步说明，AI 本身理解操作方式
- 保留：关键约束值、异常处理提示、易错点

## Rule vs Skill 选择

| 用 Rule | 用 Skill |
|---------|---------|
| 每条 bash 都要遵守（路径规范） | 特定任务才需要（Python 环境） |
| 每次工具调用都要遵守（写入限制） | 特定场景才触发（回滚操作） |
| 每次网络请求都要遵守（代理设置） | 参考性内容（配置路径查询） |
