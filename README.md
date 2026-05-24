# Tech Review — Session Skill Review Generator

**A Claude Code skill that analyzes your conversation and generates a beautiful dark-themed HTML knowledge summary.**

[中文](#中文说明) | [English](#english)

---

## English

### What It Does

After a coding or problem-solving session with Claude Code, run `/tech-review` to automatically:

1. **Extract skills** — Identifies technical skills, strategies, design patterns, and thinking models used in the conversation
2. **Generate HTML report** — Creates a polished dark-themed HTML page with categorized skill cards, difficulty ratings, code snippets, and knowledge graphs
3. **Update index** — Maintains a master index with timeline view, skill aggregation, tag cloud, and a GitHub-style learning heatmap

### Features

- 5 skill categories: Tech / Strategy / Design / Business / Thinking
- Difficulty (1-5 stars) and importance (1-5 flames) ratings for each skill
- Syntax-highlighted code snippets via Prism.js
- Knowledge relationship mapping between skills
- Editable "My Notes" area on each skill card
- Print-friendly mode with automatic light theme
- Auto-aggregated skill index tracking your growth over time

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

Output is saved to `mytech/` in your current working directory.

### Output Structure

```
mytech/
  index.html                          # Master index with 3 views
  session_2025_05_20_api_design.html  # Session report
  session_2025_05_22_react_app.html   # Session report
  ...
```

### License

MIT

---

## 中文说明

### 这是什么

一个 Claude Code 技能插件——每次编程或解决问题的对话结束后，运行 `/tech-review`，自动分析对话中用到的所有技能、策略和思维方式，生成一份精美的深色主题 HTML 知识复盘文件。

### 核心功能

- 自动识别对话中的技术技能、策略、设计模式、商业思维、思维模型
- 生成深色主题 HTML 报告，每个知识点含难度评级、重要性评级、代码片段、关联图谱
- 自动维护索引页面：时间线视图、技能聚合、标签云、GitHub 风格学习热力图
- 可编辑的"我的笔记"区域，支持打印导出 PDF

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

生成的文件保存在当前工作目录的 `mytech/` 文件夹下。
