# Tech Review — Session Skill Review Generator

**A Claude Code skill that analyzes your conversation and generates a beautiful dark-themed HTML knowledge summary.**

[中文](#中文说明) | [English](#english)

---

## English

### Why This Exists

We ask AI to solve problems every day — and it delivers. But here's the thing: most of the time, we walk away with the answer and never look back at *how* AI got there. What knowledge did it draw on? What strategies did it choose? What thinking patterns made the solution work?

That's a missed opportunity. Every conversation with AI is packed with skills, patterns, and decision-making approaches that you could internalize — if only someone laid them out for you.

**Tech Review** does exactly that. After each conversation, it reviews everything that happened and extracts the knowledge and strategies AI applied — the ones you might not have noticed. Over time, this builds into a personal knowledge base that grows with you: you learn faster, think more clearly, and communicate with AI far more effectively.

The goal isn't to replace your own thinking — it's to make sure you actually *learn* from every interaction, not just consume the output.

### What It Does

After a coding or problem-solving session with Claude Code, run `/tech-review` to automatically:

1. **Extract skills** — Identifies technical skills, strategies, design patterns, and thinking models used in the conversation
2. **Generate HTML report** — Creates a polished dark-themed HTML page with categorized skill cards, difficulty ratings, code snippets, and knowledge graphs
3. **Update index** — Maintains a master index with timeline view, skill aggregation, tag cloud, and a GitHub-style learning heatmap

### Features

- 8 skill categories: Tech / Strategy / Design / Business / Thinking / Data / Communication / AI
- Difficulty (1-5 stars) and importance (1-5 flames) ratings for each skill
- Self-contained syntax-highlighted code snippets (no external CDN)
- Knowledge relationship mapping between skills
- Editable "My Notes" area on each skill card
- Print-friendly mode with automatic light theme
- Clickable tag cloud and skill entries — jump directly to knowledge details
- Auto-aggregated skill index tracking your growth over time
- **Global knowledge base** — all sessions stored in `~/mytech/` regardless of project directory, automatically initialized on first run

### Installation

Copy the `SKILL.md` file into your Claude Code skills directory:

```bash
# macOS / Linux
mkdir -p ~/.claude/skills/tech-review
cp SKILL.md ~/.claude/skills/tech-review/

# Windows
mkdir %USERPROFILE%\.claude\skills\tech-review
copy SKILL.md %USERPROFILE%\.claude\skills\tech-review\
```

### Usage

In any Claude Code conversation, trigger the skill with:

- `/tech-review`
- Natural language: "tech review", "summarize skills", "learning summary"

Output is saved to `~/mytech/` (your home directory), forming a global knowledge base across all projects.

### Output Structure

```
~/mytech/
  index.html                          # Master index with 3 views
  session_2025_05_20_api_design.html  # Session report
  session_2025_05_22_react_app.html   # Session report
  ...
```

### License

MIT

---

## 中文说明

### 设计理念

我们每天都在用 AI 解决问题，而且它确实很能干。但问题是：大多数时候，我们拿到结果就走了，从来没回头看过 AI 是*怎么做到的*。它调用了什么知识？选择了什么策略？背后的思维模式是什么？

这其实是一种浪费。每一次与 AI 的对话里，都藏着大量你可以内化的技能、模式和决策方法——只是没人帮你把它们摊开来看。

**Tech Review** 做的就是这件事。每次对话结束后，它会回顾整个过程，把 AI 运用到的知识和策略提取出来——那些你可能根本没注意到的东西。日积月累，这些复盘会汇聚成你自己的知识体系：你学得更快、想得更清楚，也能更高效地与 AI 协作。

目标不是替代你的思考，而是确保每一次与 AI 的交互，你都真正学到了东西，而不只是拿走了一个答案。

### 这是什么

一个 Claude Code 技能插件——每次编程或解决问题的对话结束后，运行 `/tech-review`，自动分析对话中用到的所有技能、策略和思维方式，生成一份精美的深色主题 HTML 知识复盘文件。

### 核心功能

- 8 大知识分类：技术 / 策略 / 设计 / 商业 / 思维 / 数据 / 沟通 / AI
- 每个知识点含难度评级、重要性评级、代码片段、关联图谱
- 纯内联代码高亮，无需外部 CDN（国内友好）
- 词云标签和技能条目可点击跳转到具体知识详情
- 可编辑的"我的笔记"区域，支持打印导出 PDF
- **全局知识库** — 所有 session 存储在 `~/mytech/`，不限于单个项目目录，首次运行自动初始化
- 自动维护索引页面：时间线视图、技能聚合、标签云、八维雷达图、GitHub 风格学习热力图

### 安装

将 `SKILL.md` 复制到 Claude Code 技能目录：

```powershell
mkdir $env:USERPROFILE\.claude\skills\tech-review
copy SKILL.md $env:USERPROFILE\.claude\skills\tech-review\
```

### 使用方式

在任意 Claude Code 对话中：

- 输入 `/tech-review`
- 或自然语言：`技能复盘`、`总结技能`、`复盘一下`、`总结下学到了什么`

生成的文件保存在用户主目录的 `mytech/` 文件夹下（`~/mytech/`），形成跨项目的全局知识图谱。
