# 小红书系列帖：Claude Code Token 优化体系

> GitHub 仓库链接：https://github.com/YonganZhang/claude-code-token-optimization

---

## 帖 1：钩子帖（引流）

**标题：** Claude Code 用了一个月，我把 Token 消耗砍了 60%

**封面建议：** 左右对比图 —— 左边「优化前」200行 CLAUDE.md 截图（红色标注），右边「优化后」60行 + 3个小 Skill 文件（绿色标注）

**正文：**

用 Claude Code 写代码一个月，发现一个问题：Token 消耗太快了。

研究了一下原因：CLAUDE.md、Rules 这些配置文件，每一轮对话都会全文加载。200 行的 CLAUDE.md，聊 20 轮就是 4000 行重复加载。

于是我设计了一套分层优化体系：

把信息按「是否每轮都需要」分成四层：
- L0 Rule（≤10行）：每轮加载，只放触发索引
- L1 Skill 索引（≤50行）：触发时才加载
- L2 详情文件：AI 按需读取
- L3 源码：AI 直接读

效果：固定成本从 200+ 行降到 60 行，省了 60% 以上。

完整方案开源在 GitHub（评论区链接），包含示例文件可以直接复制用。

**标签：** #ClaudeCode #AI编程 #Token优化 #开发效率 #程序员日常

---

## 帖 2：四层架构

**标题：** Claude Code 四层渐进式披露：让 AI 只看它需要的信息

**封面建议：** 四层金字塔图 —— 从上到下 L0→L3，每层标注载体、行数限制、加载时机。越往上越小（固定成本），越往下越大（按需成本）

**正文：**

Claude Code 的上下文窗口是有限的。塞太多信息进去，不仅浪费 Token，还会让 AI 注意力分散。

我的解决方案：四层渐进式披露。

第一层 L0：Rule
每轮都加载，所以必须精简到 10 行以内。只放触发条件，比如「创建 CLAUDE.md 时 → 先读 skill xxx」。

第二层 L1：Skill 索引
触发时才加载。SKILL.md 控制在 50 行以内，只放目录和摘要。

第三层 L2：详情文件
放在 Skill 的 references/ 目录下。AI 需要时用 Read 工具按需读取。

第四层 L3：源码
不存储，AI 直接读源文件。

核心思想：每一层只在需要时才加载下一层。

这套体系的 Skill 文件本身就是用这个模式组织的 —— 吃自己的狗粮。

GitHub 链接见评论区。

**标签：** #ClaudeCode #渐进式披露 #上下文管理 #AI编程 #开发者工具

---

## 帖 3：Rule + Skill 联动

**标题：** Rule 只写 10 行就够了：Claude Code 的 Rule + Skill 联动模式

**封面建议：** 流程图 —— environment.md（5行）→ 箭头 → project-doc-guide Skill（53行）→ 箭头 → 生成 CLAUDE.md + 项目 Skills

**正文：**

很多人的 Claude Code Rule 写了几十行，其实大部分内容不需要每轮加载。

我的做法：Rule 只写触发索引，详情放 Skill。

举个例子，我的 environment.md 只有 5 行：

```
# Environment
- IDE: VSCode with Claude extension
- 项目文档/初始化项目 → 先读 skill project-doc-guide
- 创建/修改 Rule 或 Skill → 先读 skill skill-creator
- 任务完成后新增模块 → 批量更新项目索引
```

每轮固定成本：5 行。
AI 只在匹配到「初始化项目」时才加载 project-doc-guide 的 53 行详细规则。

对比一下：
- 不联动：53 行 × 每轮 = 高成本
- 联动后：5 行 × 每轮 + 53 行 × 偶尔 = 低成本

创建 Rule 的原则：超过 10 行就该拆。触发条件留 Rule，详情放 Skill。

**标签：** #ClaudeCode #Rule #Skill #Token节省 #AI配置

---

## 帖 4：CLAUDE.md 瘦身

**标题：** 你的 CLAUDE.md 太胖了：从 200 行瘦身到 60 行的实战

**封面建议：** 三栏对比 —— 左边「原始 CLAUDE.md 200行」，中间箭头「拆分」，右边「CLAUDE.md 60行 + project-modules Skill + project-data Skill」

**正文：**

CLAUDE.md 是 Claude Code 的项目说明书，每轮对话全文加载。

问题是：大部分内容 AI 不是每轮都需要。技术栈版本、模块列表、配置文件路径……这些是查阅型信息，不是规范约束。

我的拆分原则：

留在 CLAUDE.md 的（每轮加载）：
- 项目身份（一句话）
- 项目独有的代码规范
- 常用开发命令
- 操作约束

拆到项目 Skill 的（按需加载）：
- project-modules：模块名 → 职责 → 入口 → 关键导出
- project-data：配置文件 → 位置 → 用途
- project-pipelines：构建流程 → 配置 → 步骤

判断标准很简单：删掉这段内容后，AI 还能正确写代码吗？能 → 拆出去。

实测效果：CLAUDE.md 从 200 行降到 60 行，20 轮对话省了 2800 行重复加载。

完整方案和示例文件：GitHub 链接见评论区。

**标签：** #ClaudeCode #CLAUDE_md #项目配置 #Token优化 #AI编程效率
