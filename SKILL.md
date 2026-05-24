---
name: tech-review
description: "Analyze the current conversation to extract technical skills, strategies, and design patterns used, then generate a beautiful dark-themed HTML knowledge summary with an auto-updated index. Trigger: /tech-review, '技能复盘', '总结技能', 'tech review', '复盘一下', '总结下学到了什么'."
---

# Tech Review — 对话技能复盘生成器

你的任务是分析当前对话，提取所有运用到的技能、策略和思维方式，生成一份精美的深色主题HTML知识总结文件，并更新索引。

## 触发条件

当用户表达以下意图时自动触发：
- 直接命令：`/tech-review`
- 中文表达：`总结技能`、`技能复盘`、`复盘一下`、`总结下学到了什么`
- 英文表达：`tech review`、`summarize skills`、`learning summary`
- 相关意图：生成学习总结HTML、回顾本次对话用到的策略/技术

## 输出目录

默认保存到当前工作目录下的 `mytech/` 文件夹。如果该目录不存在则自动创建。

> 用户可通过参数指定其他路径，例如 `/tech-review --output ~/notes/tech/`

## 工作流程

### Step 1: 分析对话内容

回顾整个对话历史，识别以下类别的技能和策略：

- **技术 (tech)** — 具体的技术工具、库、API、算法、编程模式
- **策略 (strategy)** — 问题解决策略、架构决策、多方案选择的方法论
- **设计 (design)** — UI/UX设计原则、信息架构、视觉传达技巧
- **商业 (business)** — 商业策略、定价、运营优化、市场分析
- **思维 (thinking)** — 思维模型、分析框架、认知心理学应用

对每个识别到的技能，提取：
1. **名称** — 简洁的技能名称（中文+英文术语）
2. **类别** — tech / strategy / design / business / thinking
3. **难度** — 1-5星（⭐），1=入门概念，3=中级实践，5=高级架构
4. **重要性** — 1-5星（🔥），评估该技能的通用价值
5. **WHAT** — 这个技能/策略是什么，用通俗的语言解释
6. **HOW** — 在本次对话中具体怎么用的，举实际例子。如果涉及代码，提取关键代码片段（不超过10行），用 `<pre><code>` 标签包裹
7. **SOURCE** — 这个思想/技术的来源（书籍、学科、发明人等）
8. **关联** — 与本session中其他技能的关系（如"依赖于 #s2"、"是 #s5 的上层策略"）

### Step 2: 确定主题和文件名

- 从对话内容推断一个简短的项目主题（英文，用下划线连接）
- 文件名格式：`session_YYYY_MM_DD_<topic>.html`
- 保存路径：输出目录（默认为当前工作目录下的 `mytech/`）
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

**深浅交替背景（最重要的改进）：**

每个知识点是一个 `.skill-section`，通过奇偶行交替背景色来自然分隔：

```css
.skill-section{margin-bottom:0;padding:40px 32px;border-left:4px solid transparent}
.skill-section:nth-of-type(odd){background:rgba(30,41,59,.5)}
.skill-section:nth-of-type(even){background:rgba(15,23,42,.95)}
```

这样相邻知识点有明显的深/浅色差，用户无需额外思考就能感知分界。

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
```

**Section编号颜色也跟随类别：**

```css
[data-cat="strategy"] .section-num{color:#fbbf24;background:rgba(251,191,36,.1)}
[data-cat="tech"] .section-num{color:#38bdf8;background:rgba(56,189,248,.1)}
/* ... 其他类别同理 */
```

**颜色体系（深色主题）：**
- 背景：`#0f172a`
- 卡片：`rgba(255,255,255,.03)`（比背景稍亮，不再用硬编码#1e293b，避免和交替背景冲突）
- 技术(tech)：`#38bdf8`（蓝）
- 策略(strategy)：`#fbbf24`（金）
- 设计(design)：`#f472b6`（粉）
- 商业(business)：`#34d399`（绿）
- 思维(thinking)：`#a78bfa`（紫）

**Badge样式：**
```css
.badge-tech{background:rgba(56,189,248,.15);color:#38bdf8}
.badge-strategy{background:rgba(251,191,36,.15);color:#fbbf24}
.badge-design{background:rgba(244,114,182,.15);color:#f472b6}
.badge-business{background:rgba(52,211,153,.15);color:#34d399}
.badge-thinking{background:rgba(167,139,250,.15);color:#a78bfa}
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

不使用任何外部CDN（国内访问慢），改为内联CSS+轻量JS实现代码着色。

CSS部分（放在 `<style>` 中）：
```css
/* Code blocks — self-contained, no external CDN */
pre{background:rgba(0,0,0,.35);border-radius:8px;
  border:1px solid rgba(255,255,255,.08);margin:12px 0;padding:16px 20px;
  overflow-x:auto;line-height:1.5}
pre code{font-family:'Fira Code',Consolas,'Courier New',monospace;font-size:13px;
  color:#d4d4d4;white-space:pre}
code{font-family:'Fira Code',Consolas,'Courier New',monospace;font-size:.9em;
  background:rgba(255,255,255,.06);padding:1px 5px;border-radius:4px}
pre code{background:none;padding:0}
/* Inline syntax colors */
.hl-kw{color:#c792ea}.hl-str{color:#c3e88d}.hl-cm{color:#546e7a;font-style:italic}
.hl-fn{color:#82aaff}.hl-num{color:#f78c6c}.hl-op{color:#89ddff}
```

HOW部分的代码使用：
```html
<pre><code>
// 关键代码片段
var offset = 0.004;
var angle = (2 * Math.PI * j) / g.length;
</code></pre>
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

每个 `.skill-section` 中，在"我的笔记"上方或下方加入关联提示：

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

页面底部添加一个简单的关联关系总览（文字版，非图表，保持轻量）：

```html
<div class="relations-overview">
  <h2>🔗 知识关联图谱</h2>
  <div class="relation-map">
    <div class="rel-chain">
      <span class="rel-node" style="border-color:#fbbf24">多策略容错</span>
      <span class="rel-arrow">→</span>
      <span class="rel-node" style="border-color:#38bdf8">地理编码</span>
      <span class="rel-arrow">→</span>
      <span class="rel-node" style="border-color:#38bdf8">坐标系渲染</span>
    </div>
    <!-- 更多关联链 -->
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
  pre[class*="language-"]{background:#f5f5f5!important;border:1px solid #ddd!important;
    color:#333!important}
  h1,h2,h3{color:#1a1a1a!important}
  .badge-tech,.badge-strategy,.badge-design,.badge-business,.badge-thinking{
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
  <!-- 步骤间用CSS ::after箭头连接 -->
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

生成session文件后，**必须更新**输出目录中的 `index.html` 里的 `sessions` 数组。

索引文件使用JavaScript数据驱动渲染，只需往 `var sessions = [...]` 数组中追加一个新条目：

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

如果索引文件不存在，则根据以下规格自动创建。索引文件包含三个视图：
- **按时间**：展开可看每个session包含的知识点列表，点击知识点名称跳转（新窗口+锚点定位）
- **按技能**：自动聚合所有session中出现过的同名技能，显示频次 + 标签云词频可视化
  - 出现1次：`×1`（灰色）
  - 出现2次：`巩固 ×2`（金色）
  - 出现3+次：`熟练 ×3`（绿色）
- **统计仪表盘**：技能雷达图、类别分布条、成长时间线（堆叠柱状图）、学习热力图（GitHub风格）

这个设计自动处理了知识点重复的问题——同一个技能在不同项目中的应用会被自动归集，频次越高说明越熟练。

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

## 重要提醒

这个skill生成的是学习资料，不是工作交付物。语气应该像一位耐心的导师在讲解，而不是一份干巴巴的技术报告。让读者读完之后觉得"原来如此！"而不是"这是什么？"。
