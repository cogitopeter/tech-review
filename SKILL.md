---
name: tech-review
description: "Session skill review and knowledge summary generator. Use this skill whenever the user asks to summarize skills, review what was learned, do a tech review, or says things like '总结技能', '技能复盘', 'tech review', '复盘一下', '总结下学到了什么'. Also trigger when the user mentions generating a learning summary HTML, or asks what strategies/techniques were used in the conversation. This skill analyzes the entire conversation to extract technical skills, strategies, design patterns, and problem-solving approaches that were used, then generates a beautiful dark-themed HTML knowledge file and updates the knowledge index."
---

# Tech Review — 对话技能复盘生成器

你的任务是分析当前对话，提取所有运用到的技能、策略和思维方式，生成一份精美的深色主题HTML知识总结文件，并更新索引。

## 工作流程

### Step 0: 初始化知识库目录

**路径检测（跨平台）：**

通过 shell 命令获取用户主目录，确定知识库路径：
- Windows: `$env:USERPROFILE\mytech\`
- macOS/Linux: `$HOME/mytech/`

**首次运行初始化：**

检查知识库目录是否存在：
- 如果目录不存在 → 创建目录 + 生成初始 `index.html`（使用本文件末尾的 INDEX_TEMPLATE）
- 如果目录已存在 → 直接读取已有 `session_*.html` 文件计数，Session编号 = 已有文件数 + 1

**重要：** 不要硬编码任何绝对路径。所有路径都基于用户主目录动态拼接。

### Step 1: 分析对话内容

回顾整个对话历史，识别以下类别的技能和策略：

- **技术 (tech)** — 具体的技术工具、库、API、算法、编程模式
- **策略 (strategy)** — 问题解决策略、架构决策、多方案选择的方法论
- **设计 (design)** — UI/UX设计策略、信息架构、交互逻辑、用户体验决策（注意：不包括纯CSS实现、视觉样式细节）
- **商业 (business)** — 商业策略、定价、运营优化、市场分析
- **思维 (thinking)** — 思维模型、分析框架、认知心理学应用
- **数据 (data)** — 数据分析、数据清洗、可视化、统计方法、数据管道
- **沟通 (communication)** — 文档写作、演示技巧、说服力表达、信息传达策略
- **AI (ai)** — AI/LLM应用、提示工程、模型选择、AI工作流设计

对每个识别到的技能，提取：
1. **名称** — 简洁的技能名称（中文+英文术语）
2. **类别** — tech / strategy / design / business / thinking / data / communication / ai
3. **难度** — 1-5星（⭐），1=入门概念，3=中级实践，5=高级架构
4. **重要性** — 1-5星（🔥），评估该技能的通用价值
5. **WHAT** — 这个技能/策略是什么，用通俗的语言解释
6. **HOW** — 在本次对话中具体怎么用的，举实际例子。如果涉及代码，提取关键代码片段（不超过10行），用 `<pre><code>` 标签包裹
7. **SOURCE** — 这个思想/技术的来源（书籍、学科、发明人等）
8. **关联** — 与本session中其他技能的关系（如"依赖于 #s2"、"是 #s5 的上层策略"）

**design 分类的特殊规则：**
- 页面本身的设计元素（排版样式、CSS技巧、配色方案、动画效果等）是实现手段，**不作为技能点提取**
- 只有上升到设计策略层面的内容才归入 design：信息架构设计、交互逻辑、用户体验决策、数据可视化的表达策略等
- 如果一个设计决策背后有明确的用户体验原理（如格式塔原则、认知负荷理论），可以提取为 design 或 thinking 类别

### Step 2: 确定主题和文件名

- 从对话内容推断一个简短的项目主题（英文，用下划线连接）
- 文件名格式：`session_YYYY_MM_DD_<topic>.html`
- 保存路径：`<用户主目录>/mytech/`
- 检查目录下已有的 `session_*.html` 文件数量，Session编号 = 已有文件数 + 1

### Step 3: 生成HTML文件

使用下面的模板结构和样式生成HTML。

#### 页面结构
1. **Header** — Session编号、主题标签、日期
2. **目录 (TOC)** — 所有技能的快速导航
3. **技能章节** — 每个技能一个 `.skill-section`，内含卡片、流程图、洞察框等
4. **用户笔记区** — 每个技能卡片下方留"我的笔记"区域（contenteditable）
5. **关联图** — 页面底部展示知识点间的关联关系
6. **统计摘要** — 技能数量、难度分布、产出文件数等
7. **延伸阅读** — 推荐相关书籍/资源

#### 核心设计规范

**深浅交替背景：**

每个知识点是一个 `.skill-section`，通过奇偶行交替背景色来自然分隔：

```css
.skill-section{margin-bottom:0;padding:40px 32px;border-left:4px solid transparent}
.skill-section:nth-of-type(odd){background:rgba(30,41,59,.5)}
.skill-section:nth-of-type(even){background:rgba(15,23,42,.95)}
```

**类别色带（左边框）：**

每个 `.skill-section` 必须添加 `data-cat` 属性，CSS根据它设置左边框颜色：

```html
<div class="skill-section" id="s1" data-cat="strategy">
```

```css
.skill-section[data-cat="strategy"]{border-left-color:#fbbf24}
.skill-section[data-cat="tech"]{border-left-color:#38bdf8}
.skill-section[data-cat="design"]{border-left-color:#f472b6}
.skill-section[data-cat="business"]{border-left-color:#34d399}
.skill-section[data-cat="thinking"]{border-left-color:#a78bfa}
.skill-section[data-cat="data"]{border-left-color:#fb923c}
.skill-section[data-cat="communication"]{border-left-color:#e879f9}
.skill-section[data-cat="ai"]{border-left-color:#22d3ee}
```

**Section编号颜色也跟随类别：**

```css
[data-cat="strategy"] .section-num{color:#fbbf24;background:rgba(251,191,36,.1)}
[data-cat="tech"] .section-num{color:#38bdf8;background:rgba(56,189,248,.1)}
[data-cat="design"] .section-num{color:#f472b6;background:rgba(244,114,182,.1)}
[data-cat="business"] .section-num{color:#34d399;background:rgba(52,211,153,.1)}
[data-cat="thinking"] .section-num{color:#a78bfa;background:rgba(167,139,250,.1)}
[data-cat="data"] .section-num{color:#fb923c;background:rgba(251,147,60,.1)}
[data-cat="communication"] .section-num{color:#e879f9;background:rgba(232,121,249,.1)}
[data-cat="ai"] .section-num{color:#22d3ee;background:rgba(34,211,238,.1)}
```

**颜色体系（深色主题）：**
- 背景：`#0f172a`
- 卡片：`rgba(255,255,255,.03)`
- 技术(tech)：`#38bdf8`（蓝）
- 策略(strategy)：`#fbbf24`（金）
- 设计(design)：`#f472b6`（粉）
- 商业(business)：`#34d399`（绿）
- 思维(thinking)：`#a78bfa`（紫）
- 数据(data)：`#fb923c`（橙）
- 沟通(communication)：`#e879f9`（亮紫）
- AI(ai)：`#22d3ee`（青）

**Badge样式：**
```css
.badge-tech{background:rgba(56,189,248,.15);color:#38bdf8}
.badge-strategy{background:rgba(251,191,36,.15);color:#fbbf24}
.badge-design{background:rgba(244,114,182,.15);color:#f472b6}
.badge-business{background:rgba(52,211,153,.15);color:#34d399}
.badge-thinking{background:rgba(167,139,250,.15);color:#a78bfa}
.badge-data{background:rgba(251,147,60,.15);color:#fb923c}
.badge-communication{background:rgba(232,121,249,.15);color:#e879f9}
.badge-ai{background:rgba(34,211,238,.15);color:#22d3ee}
```

**难度/重要性标记：**

每个技能标题旁显示难度星和重要性火焰：

```html
<div class="skill-meta">
  <span class="meta-item" title="难度"><span class="meta-label">难度</span> ⭐⭐⭐</span>
  <span class="meta-item" title="重要性"><span class="meta-label">重要</span> 🔥🔥🔥🔥</span>
  <span class="meta-item relation-link" title="关联知识点">↔ #s2 地理编码</span>
</div>
```

```css
.skill-meta{display:flex;gap:16px;flex-wrap:wrap;margin:8px 0 16px;font-size:13px}
.meta-item{padding:3px 10px;background:rgba(255,255,255,.04);border-radius:8px;color:var(--text2)}
.meta-label{font-size:11px;margin-right:4px}
.relation-link{cursor:pointer;color:var(--accent);border:1px solid rgba(56,189,248,.2)}
.relation-link:hover{background:rgba(56,189,248,.1)}
```

**代码高亮（纯内联无CDN）：**

不使用任何外部CDN，改为内联CSS+轻量JS实现代码着色。

CSS部分（放在 `<style>` 中）：
```css
pre{background:rgba(0,0,0,.35);border-radius:8px;
  border:1px solid rgba(255,255,255,.08);margin:12px 0;padding:16px 20px;
  overflow-x:auto;line-height:1.5}
pre code{font-family:'Fira Code',Consolas,'Courier New',monospace;font-size:13px;
  color:#d4d4d4;white-space:pre}
code{font-family:'Fira Code',Consolas,'Courier New',monospace;font-size:.9em;
  background:rgba(255,255,255,.06);padding:1px 5px;border-radius:4px}
pre code{background:none;padding:0}
.hl-kw{color:#c792ea}.hl-str{color:#c3e88d}.hl-cm{color:#546e7a;font-style:italic}
.hl-fn{color:#82aaff}.hl-num{color:#f78c6c}.hl-op{color:#89ddff}
```

在 `</body>` 前放一段轻量内联语法着色脚本：
```html
<script>
(function(){
  document.querySelectorAll('pre code').forEach(function(el){
    var h=el.innerHTML;
    h=h.replace(/(#[^\n]*)/g,'<span class="hl-cm">$1</span>');
    h=h.replace(/(&quot;[^&]*?&quot;|'[^']*?'|"[^"]*?")/g,'<span class="hl-str">$1</span>');
    h=h.replace(/\b(var|let|const|function|return|if|else|for|while|new|this|class|import|export|from|async|await|try|catch|def|self|with|as|yield|True|False|None|mkdir|cd|git|npm|curl|pip|install|echo|sudo)\b/g,'<span class="hl-kw">$1</span>');
    h=h.replace(/\b(\d+\.?\d*)\b/g,'<span class="hl-num">$1</span>');
    el.innerHTML=h;
  });
})();
</script>
```

**关联知识点：**

每个 `.skill-section` 中，在"我的笔记"上方加入关联提示：

```html
<div class="relations">
  <span class="rel-label">↔ 关联：</span>
  <a href="#s2" class="rel-link">地理编码 (Geocoding)</a>
  <a href="#s10" class="rel-link">环境适配</a>
</div>
```

```css
.relations{margin:12px 0;padding:8px 12px;background:rgba(255,255,255,.02);
  border-radius:8px;font-size:12px;color:var(--text2)}
.rel-label{margin-right:8px}
.rel-link{color:var(--accent);text-decoration:none;margin-right:12px}
.rel-link:hover{text-decoration:underline}
```

页面底部添加关联关系总览：

```html
<div class="relations-overview">
  <h2>🔗 知识关联图谱</h2>
  <div class="relation-map">
    <div class="rel-chain">
      <span class="rel-node" style="border-color:#fbbf24">多策略容错</span>
      <span class="rel-arrow">→</span>
      <span class="rel-node" style="border-color:#38bdf8">地理编码</span>
    </div>
  </div>
</div>
```

**导出/打印优化：**

在页面右上角固定一个打印按钮：

```html
<button class="print-btn" onclick="window.print()" title="打印/导出PDF">🖨️</button>
```

```css
.print-btn{position:fixed;top:16px;right:16px;width:40px;height:40px;border-radius:50%;
  border:1px solid rgba(255,255,255,.15);background:rgba(30,41,59,.9);color:var(--text);
  font-size:18px;cursor:pointer;z-index:100;transition:all .2s;display:flex;align-items:center;justify-content:center}
.print-btn:hover{background:rgba(56,189,248,.2);border-color:var(--accent)}
```

打印时自动切换浅色主题：

```css
@media print{
  body{background:#fff!important;color:#1a1a1a!important;font-size:11pt}
  .skill-section{background:#fff!important;border-left-width:3px!important;
    page-break-inside:avoid;padding:20px 16px!important}
  .skill-section:nth-of-type(odd){background:#f8f9fa!important}
  .print-btn,.my-notes,[contenteditable]{display:none!important}
  pre{background:#f5f5f5!important;border:1px solid #ddd!important;color:#333!important}
  h1,h2,h3{color:#1a1a1a!important}
  .badge-tech,.badge-strategy,.badge-design,.badge-business,.badge-thinking,
  .badge-data,.badge-communication,.badge-ai{
    border:1px solid #ccc!important;background:#f0f0f0!important;color:#333!important}
  .section-num{color:inherit!important;background:#eee!important}
  a{color:#0066cc!important}
}
```

**用户笔记区：**
```html
<div class="notes-label">&#9998; 我的笔记</div>
<div class="my-notes" contenteditable="true" 
     data-placeholder="点击此处记录你的理解和想法..."></div>
```

**流程图（用于多步骤策略）：**
```html
<div class="flow">
  <div class="flow-step">
    <div class="step-num">Step 1</div>
    <div class="step-title">标题</div>
    <div class="step-desc">描述</div>
  </div>
</div>
```

**洞察框（用于核心原则）：**
```html
<div class="insight-box">
  <h3>&#128161; 核心思想</h3>
  <ul><li>要点...</li></ul>
</div>
```

### Step 4: 更新索引文件

生成session文件后，**必须更新** `<用户主目录>/mytech/index.html` 中的 `sessions` 数组。

往 `var sessions = [...]` 数组中追加一个新条目：

```javascript
{
  id: <新编号>,
  date: 'YYYY-MM-DD',
  topic: '项目主题（中文）',
  title: '简短描述',
  file: 'session_YYYY_MM_DD_topic.html',
  skills: [
    {name:'技能名称 (English Term)', cat:'tech', brief:'一句话描述应用场景', anchor:'s1'},
    // ... 每个技能一条，anchor对应session HTML中 .skill-section 的 id
  ]
}
```

索引文件有三个视图：
- **按时间**：展开可看每个session包含的知识点列表，点击知识点名称跳转（新窗口+锚点定位）
- **按技能**：自动聚合所有session中出现过的同名技能，显示频次 + 标签云词频可视化
  - 出现1次：`×1`（灰色）
  - 出现2次：`巩固 ×2`（金色）
  - 出现3+次：`熟练 ×3`（绿色）
- **统计仪表盘**：技能雷达图、类别分布条、成长时间线（堆叠柱状图）、学习热力图（GitHub风格）

### Step 5: 展示并确认

生成文件后：
1. 列出识别到的技能清单（编号+名称+类别+难度⭐+重要性🔥）
2. 告知session文件保存路径
3. 告知索引文件已更新
4. 提醒用户可以在浏览器打开后在"我的笔记"区域补充想法
5. 提醒可以用打印按钮导出PDF（自动切换浅色主题）

## 质量要求

- **内容深度**：不要只列名词，要解释WHY——为什么这个策略有效，背后的原理是什么
- **通俗易懂**：假设读者是聪明但不一定是技术专家的人，用类比和例子帮助理解
- **实际联系**：HOW部分必须引用本次对话中的具体案例，不能泛泛而谈
- **代码实例**：HOW部分如果涉及代码实现，提取最关键的代码片段（3-10行），用 `<pre><code class="language-xxx">` 包裹
- **来源可信**：SOURCE引用真实的书籍、学科、人物，不要编造
- **语言**：全部使用简体中文，技术术语附英文原文
- **数量**：通常一次对话能提取6-15个技能点，不要为凑数而注水，也不要遗漏重要的
- **视觉分隔**：确保每个知识点的 `.skill-section` 都有 `data-cat` 属性，保证深浅交替和类别色带正确渲染
- **难度评估**：客观评估每个技能的难度和重要性，1-5星制
- **关联标注**：标出知识点之间的依赖、互补、层次关系
- **设计不入知识点**：页面本身的设计元素（排版、配色、CSS实现、动画效果）是呈现手段，不作为技能点提取。只有上升到设计策略层面的内容（信息架构、交互逻辑、用户体验原理）才可归入 design 类别

## 重要提醒

这个skill生成的是学习资料，不是工作交付物。语气应该像一位耐心的导师在讲解，而不是一份干巴巴的技术报告。让读者读完之后觉得"原来如此！"而不是"这是什么？"。

---

## INDEX_TEMPLATE

当 `<用户主目录>/mytech/` 目录不存在或 `index.html` 不存在时，使用以下模板生成初始 `index.html`。生成时 `var sessions = [];` 为空数组，后续每次 tech-review 会往里追加。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>知识索引 · Tech Review</title>
<style>
:root{--bg:#0f172a;--card:#1e293b;--accent:#38bdf8;--gold:#fbbf24;
  --green:#34d399;--pink:#f472b6;--purple:#a78bfa;--orange:#fb923c;
  --fuchsia:#e879f9;--cyan:#22d3ee;
  --text:#e2e8f0;--text2:#94a3b8}
*{margin:0;padding:0;box-sizing:border-box}
html{scroll-behavior:smooth}
body{font-family:-apple-system,'Microsoft YaHei','PingFang SC',sans-serif;
  background:var(--bg);color:var(--text);line-height:1.7}

.header{padding:48px 40px 32px;text-align:center;
  background:linear-gradient(135deg,#0f172a,#1e3a5f,#0f172a);
  border-bottom:1px solid rgba(56,189,248,.1)}
.header h1{font-size:28px;font-weight:800}
.header h1 span{color:var(--accent)}
.header .sub{color:var(--text2);font-size:13px;margin-top:6px}

.tab-bar{display:flex;justify-content:center;gap:4px;margin:32px auto 0;
  background:var(--card);border-radius:12px;padding:4px;max-width:500px}
.tab{flex:1;padding:8px 16px;text-align:center;border-radius:10px;
  font-size:13px;font-weight:600;cursor:pointer;color:var(--text2);transition:all .2s}
.tab.on{background:var(--accent);color:#0f172a}
.tab:hover:not(.on){color:var(--text)}

.container{max-width:1000px;margin:0 auto;padding:32px 24px}
.view{display:none}
.view.active{display:block}

.stats-bar{display:flex;gap:24px;justify-content:center;margin:24px 0 32px;flex-wrap:wrap}
.sb{text-align:center}
.sb .n{font-size:28px;font-weight:800;color:var(--accent)}
.sb .l{font-size:11px;color:var(--text2);margin-top:2px}

.session-card{background:var(--card);border-radius:14px;margin-bottom:20px;overflow:hidden;
  border:1px solid rgba(255,255,255,.05);transition:border-color .2s}
.session-card:hover{border-color:rgba(56,189,248,.2)}
.session-head{padding:18px 24px;display:flex;align-items:center;justify-content:space-between;
  cursor:pointer;user-select:none}
.session-head .left{display:flex;align-items:center;gap:14px}
.session-num{font-size:20px;font-weight:800;color:var(--accent);min-width:40px}
.session-title{font-size:15px;font-weight:700}
.session-date{font-size:12px;color:var(--text2);margin-top:2px}
.session-head .right{display:flex;align-items:center;gap:12px}
.skill-count{font-size:12px;color:var(--text2);background:rgba(255,255,255,.06);
  padding:3px 10px;border-radius:8px}
.open-btn{font-size:12px;color:var(--accent);text-decoration:none;padding:4px 12px;
  border:1px solid rgba(56,189,248,.3);border-radius:8px;transition:all .2s;white-space:nowrap}
.open-btn:hover{background:rgba(56,189,248,.1)}
.arrow{color:var(--text2);font-size:14px;transition:transform .2s;margin-left:8px}
.session-card.expanded .arrow{transform:rotate(90deg)}
.skill-list{max-height:0;overflow:hidden;transition:max-height .3s ease;
  border-top:0 solid rgba(255,255,255,.04)}
.session-card.expanded .skill-list{max-height:2000px;border-top-width:1px}
.skill-item{display:flex;align-items:center;gap:10px;padding:9px 24px;
  font-size:13px;border-bottom:1px solid rgba(255,255,255,.03);
  cursor:pointer;text-decoration:none;transition:background .15s}
.skill-item:last-child{border-bottom:none}
.skill-item:hover{background:rgba(56,189,248,.06)}
.skill-item .dot{width:8px;height:8px;border-radius:50%;flex-shrink:0}
.dot-tech{background:var(--accent)}.dot-strategy{background:var(--gold)}
.dot-design{background:var(--pink)}.dot-business{background:var(--green)}
.dot-thinking{background:var(--purple)}.dot-data{background:var(--orange)}
.dot-communication{background:var(--fuchsia)}.dot-ai{background:var(--cyan)}
.skill-item .sname{color:var(--text);transition:color .15s}
.skill-item:hover .sname{color:var(--accent)}
.skill-item .jump{font-size:10px;color:var(--accent);opacity:0;transition:opacity .15s;margin-left:4px}
.skill-item:hover .jump{opacity:1}
.skill-item .scat{font-size:10px;color:var(--text2);margin-left:auto;
  background:rgba(255,255,255,.05);padding:1px 7px;border-radius:6px}

.tag-cloud{display:flex;flex-wrap:wrap;justify-content:center;align-items:center;gap:8px 14px;
  padding:24px 20px 28px;margin-bottom:24px;
  background:rgba(255,255,255,.02);border-radius:14px;border:1px solid rgba(255,255,255,.04)}
.tag-cloud-label{width:100%;text-align:center;font-size:11px;color:var(--text2);margin-bottom:8px;
  letter-spacing:2px;text-transform:uppercase}
.tag{padding:2px 10px;border-radius:16px;cursor:pointer;transition:all .25s;
  border:1px solid transparent;white-space:nowrap;line-height:1.4}
.tag:hover{border-color:currentColor;transform:scale(1.1);filter:brightness(1.2)}

.skill-group{margin-bottom:24px}
.skill-group-head{display:flex;align-items:center;gap:10px;padding:12px 0;
  border-bottom:1px solid rgba(255,255,255,.06);margin-bottom:8px}
.skill-group-head .cat-bar{width:4px;height:22px;border-radius:2px;flex-shrink:0}
.skill-group-head h3{font-size:15px;font-weight:700;flex:1}
.freq-badge{font-size:11px;font-weight:700;padding:2px 10px;border-radius:8px}
.freq-1{background:rgba(255,255,255,.06);color:var(--text2)}
.freq-2{background:rgba(251,191,36,.12);color:var(--gold)}
.freq-3{background:rgba(52,211,153,.12);color:var(--green)}
.occurrence{display:flex;align-items:center;gap:10px;padding:6px 0 6px 14px;font-size:13px}
.occurrence .sess-link{color:var(--accent);text-decoration:none;font-size:12px}
.occurrence .sess-link:hover{text-decoration:underline}
.occurrence .context{color:var(--text2);font-size:12px;flex:1}

.cat-filters{display:flex;gap:6px;justify-content:center;margin-bottom:24px;flex-wrap:wrap}
.cf{padding:4px 14px;border-radius:20px;font-size:12px;cursor:pointer;
  border:1px solid rgba(255,255,255,.1);color:var(--text2);transition:all .2s}
.cf.on{color:#fff}
.cf[data-f="all"].on{background:rgba(255,255,255,.15);border-color:rgba(255,255,255,.3)}
.cf[data-f="tech"].on{background:rgba(56,189,248,.2);border-color:var(--accent);color:var(--accent)}
.cf[data-f="strategy"].on{background:rgba(251,191,36,.15);border-color:var(--gold);color:var(--gold)}
.cf[data-f="design"].on{background:rgba(244,114,182,.15);border-color:var(--pink);color:var(--pink)}
.cf[data-f="business"].on{background:rgba(52,211,153,.15);border-color:var(--green);color:var(--green)}
.cf[data-f="thinking"].on{background:rgba(167,139,250,.15);border-color:var(--purple);color:var(--purple)}
.cf[data-f="data"].on{background:rgba(251,147,60,.15);border-color:var(--orange);color:var(--orange)}
.cf[data-f="communication"].on{background:rgba(232,121,249,.15);border-color:var(--fuchsia);color:var(--fuchsia)}
.cf[data-f="ai"].on{background:rgba(34,211,238,.15);border-color:var(--cyan);color:var(--cyan)}

.chart-row{display:flex;gap:24px;margin-bottom:24px;flex-wrap:wrap}
.chart-card{background:var(--card);border-radius:14px;padding:24px;flex:1;min-width:300px;
  border:1px solid rgba(255,255,255,.05)}
.chart-card.full{flex-basis:100%;min-width:100%}
.chart-title{font-size:14px;font-weight:700;margin-bottom:16px;display:flex;align-items:center;gap:8px}
.chart-title .ct-icon{font-size:18px}
.chart-subtitle{font-size:11px;color:var(--text2);margin-top:-10px;margin-bottom:16px}

.radar-wrap{display:flex;justify-content:center;align-items:center}
.radar-wrap svg{width:100%;max-width:380px;height:auto}

.cat-stat-row{display:flex;align-items:center;gap:10px;margin-bottom:10px}
.cat-stat-row .cs-dot{width:10px;height:10px;border-radius:50%;flex-shrink:0}
.cat-stat-row .cs-name{font-size:13px;width:48px}
.cat-stat-row .cs-bar-bg{flex:1;height:8px;border-radius:4px;background:rgba(255,255,255,.06);overflow:hidden}
.cat-stat-row .cs-bar{height:100%;border-radius:4px;transition:width .5s ease}
.cat-stat-row .cs-count{font-size:12px;color:var(--text2);min-width:32px;text-align:right}

.timeline-wrap{overflow-x:auto;padding-bottom:4px}
.timeline-wrap svg{width:100%;min-width:400px;height:auto}

.heatmap-wrap{overflow-x:auto;padding-bottom:4px}
.heatmap-wrap svg{height:auto}
.hm-legend{display:flex;align-items:center;gap:6px;margin-top:12px;font-size:10px;color:var(--text2);justify-content:flex-end}
.hm-legend-cell{width:12px;height:12px;border-radius:2px}
.hm-tip{font-size:11px;color:var(--text2);margin-top:8px;text-align:center;font-style:italic}

.insight-row{display:flex;gap:16px;flex-wrap:wrap;margin-bottom:24px}
.ins-card{background:var(--card);border-radius:12px;padding:18px 20px;flex:1;min-width:200px;
  border:1px solid rgba(255,255,255,.05);text-align:center}
.ins-card .ins-val{font-size:24px;font-weight:800}
.ins-card .ins-label{font-size:11px;color:var(--text2);margin-top:4px}

.empty-state{text-align:center;color:var(--text2);padding:60px 20px;font-size:14px}

.footer{text-align:center;padding:32px;color:var(--text2);font-size:11px;
  border-top:1px solid rgba(255,255,255,.04);margin-top:40px}

@media(max-width:600px){
  .header{padding:36px 16px 24px}
  .header h1{font-size:22px}
  .container{padding:20px 12px}
  .session-head{padding:14px 16px;flex-direction:column;align-items:flex-start;gap:8px}
  .session-head .right{align-self:flex-end}
  .skill-item{padding:8px 16px}
  .chart-row{flex-direction:column}
  .chart-card{min-width:auto}
  .tag-cloud{gap:6px 8px;padding:16px 12px 20px}
  .insight-row{flex-direction:column}
  .ins-card{min-width:auto}
}
</style>
</head>
<body>

<div class="header">
  <h1>&#128218; 知识<span>索引</span></h1>
  <div class="sub">Tech Review &middot; 对话技能复盘汇总 &middot; 持续积累中</div>
</div>

<div class="tab-bar">
  <div class="tab on" onclick="switchView('session',this)">&#128197; 按时间</div>
  <div class="tab" onclick="switchView('skill',this)">&#128300; 按技能</div>
  <div class="tab" onclick="switchView('stats',this)">&#128202; 统计</div>
</div>

<div class="container">

<div class="stats-bar">
  <div class="sb"><div class="n" id="stat-sessions">0</div><div class="l">对话场次</div></div>
  <div class="sb"><div class="n" id="stat-skills">0</div><div class="l">知识点</div></div>
  <div class="sb"><div class="n" id="stat-unique">0</div><div class="l">独立技能</div></div>
  <div class="sb"><div class="n" id="stat-mastered">0</div><div class="l">反复出现 (2+)</div></div>
</div>

<div class="view active" id="view-session"></div>

<div class="view" id="view-skill">
  <div id="tag-cloud-container"></div>
  <div class="cat-filters">
    <div class="cf on" data-f="all" onclick="filterCat('all',this)">全部</div>
    <div class="cf" data-f="tech" onclick="filterCat('tech',this)">&#9679; 技术</div>
    <div class="cf" data-f="strategy" onclick="filterCat('strategy',this)">&#9679; 策略</div>
    <div class="cf" data-f="design" onclick="filterCat('design',this)">&#9679; 设计</div>
    <div class="cf" data-f="business" onclick="filterCat('business',this)">&#9679; 商业</div>
    <div class="cf" data-f="thinking" onclick="filterCat('thinking',this)">&#9679; 思维</div>
    <div class="cf" data-f="data" onclick="filterCat('data',this)">&#9679; 数据</div>
    <div class="cf" data-f="communication" onclick="filterCat('communication',this)">&#9679; 沟通</div>
    <div class="cf" data-f="ai" onclick="filterCat('ai',this)">&#9679; AI</div>
  </div>
  <div id="skill-groups"></div>
</div>

<div class="view" id="view-stats">
  <div class="insight-row" id="insight-row"></div>
  <div class="chart-row">
    <div class="chart-card">
      <div class="chart-title"><span class="ct-icon">&#128171;</span> 技能雷达图</div>
      <div class="chart-subtitle">八维能力画像 — 每个维度代表该类别的知识积累量</div>
      <div class="radar-wrap" id="radar-chart"></div>
    </div>
    <div class="chart-card">
      <div class="chart-title"><span class="ct-icon">&#128202;</span> 类别分布</div>
      <div class="chart-subtitle">各类别知识点数量对比</div>
      <div id="cat-bars"></div>
    </div>
  </div>
  <div class="chart-row">
    <div class="chart-card full">
      <div class="chart-title"><span class="ct-icon">&#128200;</span> 成长时间线</div>
      <div class="chart-subtitle">每次对话的知识积累 — 柱状按类别堆叠</div>
      <div class="timeline-wrap" id="timeline-chart"></div>
    </div>
  </div>
  <div class="chart-row">
    <div class="chart-card full">
      <div class="chart-title"><span class="ct-icon">&#128293;</span> 学习热力图</div>
      <div class="chart-subtitle">过去一年的学习活跃度 — 颜色越深知识点越多</div>
      <div class="heatmap-wrap" id="heatmap-chart"></div>
      <div class="hm-legend">
        <span>少</span>
        <div class="hm-legend-cell" style="background:rgba(255,255,255,.04)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.2)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.45)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.7)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.95)"></div>
        <span>多</span>
      </div>
      <div class="hm-tip">每个格子代表一天，有对话复盘的日子会亮起来</div>
    </div>
  </div>
</div>

</div>

<div class="footer">
  知识索引 &middot; 由 /tech-review skill 自动维护
</div>

<script>
var sessions = [];

var catColors = {tech:'#38bdf8',strategy:'#fbbf24',design:'#f472b6',business:'#34d399',thinking:'#a78bfa',data:'#fb923c',communication:'#e879f9',ai:'#22d3ee'};
var catLabels = {tech:'技术',strategy:'策略',design:'设计',business:'商业',thinking:'思维',data:'数据',communication:'沟通',ai:'AI'};
var catOrder = ['tech','strategy','design','business','thinking','data','communication','ai'];

var allSkills=[], uniqueNames={}, catCounts={}, uniqueCount=0, masteredCount=0;
function computeData(){
  allSkills=[]; uniqueNames={}; catCounts={};
  catOrder.forEach(function(c){catCounts[c]=0});
  sessions.forEach(function(s){
    s.skills.forEach(function(sk){
      allSkills.push(sk);
      catCounts[sk.cat]=(catCounts[sk.cat]||0)+1;
      var key=sk.name.replace(/\s*\(.*?\)\s*/g,'').trim();
      if(!uniqueNames[key]) uniqueNames[key]=[];
      uniqueNames[key].push(sk);
    });
  });
  uniqueCount=Object.keys(uniqueNames).length;
  masteredCount=0;
  for(var k in uniqueNames){if(uniqueNames[k].length>=2) masteredCount++;}
}

function render(){
  computeData();
  document.getElementById('stat-sessions').textContent=sessions.length;
  document.getElementById('stat-skills').textContent=allSkills.length;
  document.getElementById('stat-unique').textContent=uniqueCount;
  document.getElementById('stat-mastered').textContent=masteredCount;
  renderSessionView();
  renderSkillView();
  renderStatsView();
}

function renderSessionView(){
  var html='';
  sessions.slice().reverse().forEach(function(s){
    html+='<div class="session-card" onclick="toggleSession(this)">';
    html+='<div class="session-head"><div class="left">';
    html+='<div class="session-num">#'+s.id+'</div>';
    html+='<div><div class="session-title">'+s.topic+'</div>';
    html+='<div class="session-date">'+s.date+' &middot; '+s.title+'</div></div>';
    html+='</div><div class="right">';
    html+='<span class="skill-count">'+s.skills.length+' 知识点</span>';
    html+='<a class="open-btn" href="'+s.file+'" target="_blank" onclick="event.stopPropagation()">打开详情 &rarr;</a>';
    html+='<span class="arrow">&#9654;</span>';
    html+='</div></div>';
    html+='<div class="skill-list">';
    s.skills.forEach(function(sk){
      var href=s.file+(sk.anchor?'#'+sk.anchor:'');
      html+='<a class="skill-item" href="'+href+'" target="_blank" onclick="event.stopPropagation()">';
      html+='<div class="dot dot-'+sk.cat+'"></div>';
      html+='<span class="sname">'+sk.name+'</span>';
      html+='<span class="jump">&rarr;</span>';
      html+='<span class="scat">'+sk.cat+'</span>';
      html+='</a>';
    });
    html+='</div></div>';
  });
  document.getElementById('view-session').innerHTML=html;
}

function renderSkillView(){
  var grouped={}, groupOrder=[];
  sessions.forEach(function(s){
    s.skills.forEach(function(sk){
      var key=sk.name.replace(/\s*\(.*?\)\s*/g,'').trim();
      if(!grouped[key]){
        grouped[key]={name:sk.name,cat:sk.cat,occurrences:[]};
        groupOrder.push(key);
      }
      grouped[key].occurrences.push({session:s,brief:sk.brief,anchor:sk.anchor});
    });
  });
  groupOrder.sort(function(a,b){
    var diff=grouped[b].occurrences.length-grouped[a].occurrences.length;
    return diff!==0?diff:a.localeCompare(b);
  });

  var tagHtml='<div class="tag-cloud"><div class="tag-cloud-label">&#9729; 知识点词云 · 点击跳转</div>';
  var maxFreq=1;
  groupOrder.forEach(function(key){
    if(grouped[key].occurrences.length>maxFreq) maxFreq=grouped[key].occurrences.length;
  });
  groupOrder.forEach(function(key,idx){
    var g=grouped[key];
    var freq=g.occurrences.length;
    var size=Math.round(12+10*(freq/maxFreq));
    var color=catColors[g.cat]||'#94a3b8';
    var shortName=key.length>10?key.substring(0,10)+'…':key;
    tagHtml+='<span class="tag" style="font-size:'+size+'px;color:'+color+
      ';background:'+hexToRgba(color,0.08)+'" onclick="scrollToGroup(\'sg-'+idx+'\')">'+shortName+'</span>';
  });
  tagHtml+='</div>';
  document.getElementById('tag-cloud-container').innerHTML=tagHtml;

  var skillHtml='';
  groupOrder.forEach(function(key,idx){
    var g=grouped[key];
    var freq=g.occurrences.length;
    var freqClass=freq>=3?'freq-3':freq>=2?'freq-2':'freq-1';
    var freqLabel=freq>=3?'熟练 ×'+freq:freq>=2?'巩固 ×'+freq:'×'+freq;
    var catColor=catColors[g.cat]||'#94a3b8';

    skillHtml+='<div class="skill-group" id="sg-'+idx+'" data-cat="'+g.cat+'">';
    skillHtml+='<div class="skill-group-head">';
    skillHtml+='<div class="cat-bar" style="background:'+catColor+'"></div>';
    skillHtml+='<h3>'+g.name+'</h3>';
    skillHtml+='<span class="freq-badge '+freqClass+'">'+freqLabel+'</span>';
    skillHtml+='</div>';
    g.occurrences.forEach(function(o){
      var oHref=o.session.file+(o.anchor?'#'+o.anchor:'');
      skillHtml+='<div class="occurrence">';
      skillHtml+='<a class="sess-link" href="'+oHref+'" target="_blank">#'+o.session.id+' '+o.session.date+' &rarr;</a>';
      skillHtml+='<span class="context">'+o.brief+'</span>';
      skillHtml+='</div>';
    });
    skillHtml+='</div>';
  });
  document.getElementById('skill-groups').innerHTML=skillHtml||'<div class="empty-state">暂无数据</div>';
}

function renderStatsView(){
  renderInsights();
  renderRadar();
  renderCatBars();
  renderTimeline();
  renderHeatmap();
}

function renderInsights(){
  var topCat='tech', topVal=0;
  catOrder.forEach(function(c){if(catCounts[c]>topVal){topVal=catCounts[c];topCat=c;}});
  var avgSkills=sessions.length>0?(allSkills.length/sessions.length).toFixed(1):'0';
  var dates=sessions.map(function(s){return s.date}).sort();
  var span=0;
  if(dates.length>=2){
    var d0=new Date(dates[0]),d1=new Date(dates[dates.length-1]);
    span=Math.round((d1-d0)/(86400000));
  }
  var html='';
  html+='<div class="ins-card"><div class="ins-val" style="color:'+catColors[topCat]+'">'+catLabels[topCat]+'</div><div class="ins-label">最强维度</div></div>';
  html+='<div class="ins-card"><div class="ins-val" style="color:var(--accent)">'+avgSkills+'</div><div class="ins-label">平均每场知识点</div></div>';
  html+='<div class="ins-card"><div class="ins-val" style="color:var(--green)">'+span+'</div><div class="ins-label">学习跨度 (天)</div></div>';
  html+='<div class="ins-card"><div class="ins-val" style="color:var(--gold)">'+masteredCount+'</div><div class="ins-label">巩固/熟练技能</div></div>';
  document.getElementById('insight-row').innerHTML=html;
}

function renderRadar(){
  var cx=170,cy=175,r=120;
  var n=catOrder.length, step=2*Math.PI/n, start=-Math.PI/2;
  var maxVal=1;
  catOrder.forEach(function(c){if(catCounts[c]>maxVal) maxVal=catCounts[c];});

  var svg='<svg viewBox="0 0 340 350" xmlns="http://www.w3.org/2000/svg">';
  [0.25,0.5,0.75,1].forEach(function(scale){
    var pts=[];
    for(var i=0;i<n;i++){
      var a=start+i*step;
      pts.push((cx+r*scale*Math.cos(a)).toFixed(1)+','+(cy+r*scale*Math.sin(a)).toFixed(1));
    }
    svg+='<polygon points="'+pts.join(' ')+'" fill="none" stroke="rgba(255,255,255,.07)" stroke-width="1"/>';
  });
  for(var i=0;i<n;i++){
    var a=start+i*step;
    svg+='<line x1="'+cx+'" y1="'+cy+'" x2="'+(cx+r*Math.cos(a)).toFixed(1)+'" y2="'+(cy+r*Math.sin(a)).toFixed(1)+'" stroke="rgba(255,255,255,.06)" stroke-width="1"/>';
  }
  var dataPts=[];
  for(var i=0;i<n;i++){
    var a=start+i*step;
    var val=Math.max(catCounts[catOrder[i]]/maxVal,0.08);
    dataPts.push((cx+r*val*Math.cos(a)).toFixed(1)+','+(cy+r*val*Math.sin(a)).toFixed(1));
  }
  svg+='<polygon points="'+dataPts.join(' ')+'" fill="rgba(56,189,248,.1)" stroke="rgba(56,189,248,.6)" stroke-width="2"/>';
  for(var i=0;i<n;i++){
    var a=start+i*step;
    var val=Math.max(catCounts[catOrder[i]]/maxVal,0.08);
    var dx=cx+r*val*Math.cos(a);
    var dy=cy+r*val*Math.sin(a);
    var col=catColors[catOrder[i]];
    svg+='<circle cx="'+dx.toFixed(1)+'" cy="'+dy.toFixed(1)+'" r="4" fill="'+col+'" stroke="#0f172a" stroke-width="2"/>';
    var lx=cx+(r+24)*Math.cos(a);
    var ly=cy+(r+24)*Math.sin(a);
    var anchor='middle';
    if(Math.cos(a)>0.3) anchor='start';
    else if(Math.cos(a)<-0.3) anchor='end';
    svg+='<text x="'+lx.toFixed(1)+'" y="'+ly.toFixed(1)+'" text-anchor="'+anchor+'" fill="'+col+'" font-size="11" font-weight="700">'+catLabels[catOrder[i]]+'</text>';
    svg+='<text x="'+lx.toFixed(1)+'" y="'+(ly+14).toFixed(1)+'" text-anchor="'+anchor+'" fill="#94a3b8" font-size="10">'+catCounts[catOrder[i]]+' 项</text>';
  }
  svg+='</svg>';
  document.getElementById('radar-chart').innerHTML=svg;
}

function renderCatBars(){
  var maxVal=1;
  catOrder.forEach(function(c){if(catCounts[c]>maxVal) maxVal=catCounts[c];});
  var html='';
  catOrder.forEach(function(c){
    var pct=Math.round(catCounts[c]/maxVal*100);
    var col=catColors[c];
    html+='<div class="cat-stat-row">';
    html+='<div class="cs-dot" style="background:'+col+'"></div>';
    html+='<div class="cs-name">'+catLabels[c]+'</div>';
    html+='<div class="cs-bar-bg"><div class="cs-bar" style="width:'+pct+'%;background:'+col+'"></div></div>';
    html+='<div class="cs-count">'+catCounts[c]+'</div>';
    html+='</div>';
  });
  document.getElementById('cat-bars').innerHTML=html;
}

function renderTimeline(){
  var sorted=sessions.slice().sort(function(a,b){return a.id-b.id});
  var n=sorted.length;
  if(n===0){document.getElementById('timeline-chart').innerHTML='<div class="empty-state">暂无数据</div>';return;}
  var maxSkills=1;
  sorted.forEach(function(s){if(s.skills.length>maxSkills) maxSkills=s.skills.length;});
  maxSkills=Math.ceil(maxSkills/5)*5;
  var padL=40,padR=20,padT=20,padB=50;
  var barGap=n===1?0:24;
  var barW=Math.min(80,Math.max(40,(600-padL-padR-(n-1)*barGap)/n));
  var W=padL+n*barW+(n-1)*barGap+padR;
  var chartH=160;
  var H=padT+chartH+padB;
  var svg='<svg viewBox="0 0 '+W+' '+H+'" xmlns="http://www.w3.org/2000/svg">';
  for(var yi=0;yi<=4;yi++){
    var yv=Math.round(maxSkills*yi/4);
    var yy=padT+chartH-chartH*(yi/4);
    svg+='<line x1="'+padL+'" y1="'+yy+'" x2="'+(W-padR)+'" y2="'+yy+'" stroke="rgba(255,255,255,.06)"/>';
    svg+='<text x="'+(padL-6)+'" y="'+(yy+4)+'" text-anchor="end" fill="#94a3b8" font-size="10">'+yv+'</text>';
  }
  sorted.forEach(function(s,i){
    var bx=padL+i*(barW+barGap);
    var sCats={};
    catOrder.forEach(function(c){sCats[c]=0});
    s.skills.forEach(function(sk){sCats[sk.cat]=(sCats[sk.cat]||0)+1});
    var yOffset=0;
    catOrder.forEach(function(c){
      if(!sCats[c]) return;
      var segH=chartH*(sCats[c]/maxSkills);
      var sy=padT+chartH-yOffset-segH;
      svg+='<rect x="'+bx+'" y="'+sy.toFixed(1)+'" width="'+barW+'" height="'+segH.toFixed(1)+
        '" rx="3" fill="'+catColors[c]+'" opacity="0.7"><title>'+catLabels[c]+': '+sCats[c]+'</title></rect>';
      yOffset+=segH;
    });
    var topY=padT+chartH-chartH*(s.skills.length/maxSkills)-10;
    svg+='<text x="'+(bx+barW/2)+'" y="'+topY+'" text-anchor="middle" fill="var(--text)" font-size="12" font-weight="700">'+s.skills.length+'</text>';
    svg+='<text x="'+(bx+barW/2)+'" y="'+(padT+chartH+16)+'" text-anchor="middle" fill="#94a3b8" font-size="10">#'+s.id+'</text>';
    svg+='<text x="'+(bx+barW/2)+'" y="'+(padT+chartH+30)+'" text-anchor="middle" fill="#64748b" font-size="9">'+s.date.substring(5)+'</text>';
  });
  svg+='</svg>';
  document.getElementById('timeline-chart').innerHTML=svg;
}

function renderHeatmap(){
  var today=new Date();
  today.setHours(0,0,0,0);
  var start52=new Date(today);
  start52.setDate(start52.getDate()-52*7);
  var earliest=start52;
  sessions.forEach(function(s){
    var d=new Date(s.date+'T00:00:00');
    if(d<earliest) earliest=new Date(d);
  });
  var startDate=new Date(earliest);
  startDate.setDate(startDate.getDate()-startDate.getDay());
  var dateMap={};
  sessions.forEach(function(s){
    dateMap[s.date]=(dateMap[s.date]||0)+s.skills.length;
  });
  var topicMap={};
  sessions.forEach(function(s){topicMap[s.date]=s.topic;});
  var cells=[];
  var d=new Date(startDate);
  while(d<=today){
    var key=d.getFullYear()+'-'+pad2(d.getMonth()+1)+'-'+pad2(d.getDate());
    cells.push({date:key,count:dateMap[key]||0,topic:topicMap[key]||'',dow:d.getDay()});
    d.setDate(d.getDate()+1);
  }
  var cellSize=13,gap=3,cellStep=cellSize+gap;
  var weeks=Math.ceil(cells.length/7);
  var labelW=28,labelH=18;
  var W=labelW+weeks*cellStep+10;
  var H=labelH+7*cellStep+4;
  var svg='<svg viewBox="0 0 '+W+' '+H+'" width="'+W+'" xmlns="http://www.w3.org/2000/svg" style="min-width:'+W+'px">';
  var dayNames=['','一','','三','','五',''];
  for(var i=0;i<7;i++){
    if(dayNames[i]){
      svg+='<text x="'+(labelW-4)+'" y="'+(labelH+i*cellStep+cellSize-2)+'" text-anchor="end" fill="#64748b" font-size="9">'+dayNames[i]+'</text>';
    }
  }
  var lastMonth=-1;
  var cellIdx=0;
  for(var w=0;w<weeks;w++){
    for(var dow=0;dow<7;dow++){
      if(cellIdx>=cells.length) break;
      var c=cells[cellIdx];
      var cDate=new Date(c.date+'T00:00:00');
      var mon=cDate.getMonth();
      if(mon!==lastMonth && dow===0){
        var monthNames=['1月','2月','3月','4月','5月','6月','7月','8月','9月','10月','11月','12月'];
        svg+='<text x="'+(labelW+w*cellStep)+'" y="'+(labelH-5)+'" fill="#64748b" font-size="9">'+monthNames[mon]+'</text>';
        lastMonth=mon;
      }
      cellIdx++;
    }
  }
  cellIdx=0;
  for(var w=0;w<weeks;w++){
    for(var dow=0;dow<7;dow++){
      if(cellIdx>=cells.length) break;
      var c=cells[cellIdx];
      var x=labelW+w*cellStep;
      var y=labelH+dow*cellStep;
      var fill=hmColor(c.count);
      var title=c.date+(c.count>0?' · '+c.topic+' ('+c.count+'个知识点)':'');
      svg+='<rect x="'+x+'" y="'+y+'" width="'+cellSize+'" height="'+cellSize+'" rx="2" fill="'+fill+'"><title>'+title+'</title></rect>';
      cellIdx++;
    }
  }
  svg+='</svg>';
  document.getElementById('heatmap-chart').innerHTML=svg;
}

function hmColor(count){
  if(count===0) return 'rgba(255,255,255,.04)';
  if(count<=5) return 'rgba(56,189,248,.25)';
  if(count<=10) return 'rgba(56,189,248,.5)';
  if(count<=15) return 'rgba(56,189,248,.75)';
  return 'rgba(56,189,248,.95)';
}
function pad2(n){return n<10?'0'+n:''+n;}
function hexToRgba(hex,alpha){
  var r=parseInt(hex.slice(1,3),16);
  var g=parseInt(hex.slice(3,5),16);
  var b=parseInt(hex.slice(5,7),16);
  return 'rgba('+r+','+g+','+b+','+alpha+')';
}
function scrollToGroup(id){
  var el=document.getElementById(id);
  if(el){el.scrollIntoView({behavior:'smooth',block:'center'});}
}
function switchView(v,el){
  document.querySelectorAll('.tab').forEach(function(t){t.classList.remove('on')});
  el.classList.add('on');
  document.querySelectorAll('.view').forEach(function(vw){vw.classList.remove('active')});
  document.getElementById('view-'+v).classList.add('active');
}
function toggleSession(card){card.classList.toggle('expanded');}
function filterCat(cat,el){
  document.querySelectorAll('.cf').forEach(function(c){c.classList.remove('on')});
  el.classList.add('on');
  document.querySelectorAll('.skill-group').forEach(function(g){
    if(cat==='all') g.style.display='';
    else g.style.display=g.dataset.cat===cat?'':'none';
  });
}

render();
var first=document.querySelector('.session-card');
if(first) first.classList.add('expanded');
</script>
</body>
</html>
```

**注意**：以上模板中 `var sessions = [];` 为空数组。首次生成 index.html 后，每次运行 tech-review 时往该数组追加新条目即可。
