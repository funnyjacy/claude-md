# 我的 Claude Code 全局 CLAUDE.md

这是我使用 Claude Code 两年多积累下来的**全局 CLAUDE.md**（位于 `C:\Users\<用户名>\.claude\CLAUDE.md`），每次会话都会自动加载进上下文，相当于我给 Claude 的「个人使用说明书」。

## 里面有什么

- **工具链约定**：Windows 下读 PDF / Word / Excel 的最佳路径，浏览器自动化工具怎么选（agent-browser CLI vs chrome-devtools MCP），生图脚本怎么用
- **踩坑记录**：都是真实踩过的坑——Claude Code 进程内负缓存、PowerShell 5.1 中文路径乱码、MySQL 容器双重编码、KaTeX 样式未引入等，写一条少踩一次
- **协作偏好**：编辑文字先展示拟稿、commit/push 必须先确认、前端样式必须实际截图验证再交付、PR 定稿前做多视角 agent 审计
- **学习模式约定**：给 AI 学生出练习题的文件格式、难度安排原则

## 怎么用

把需要的条目抄进你自己的 `~/.claude/CLAUDE.md` 即可。注意：

- 路径里的 `<用户名>`、`<Scoop安装目录>` 等占位符要换成你自己的
- 这些规则高度个人化（比如「回答用中文」「学习模式」是我自己的需求），直接照搬不如挑有用的
- 条目来自实际踩坑反馈，比「最佳实践清单」更经得起检验

## 配套博客

写了一篇博客介绍这套东西的思路：[链接待填](https://funnyjacy.github.io/blog/)
