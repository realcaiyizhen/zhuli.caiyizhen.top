---
name: ppt-writer
description: |
  帮助用户生成高质量、视觉冲击力强的HTML演示文稿（PPT）。
  Use this skill when users ask to create a presentation, write slides, make a PPT,
  generate a deck, create a slide show, or mention "写PPT", "做演示文稿", "生成幻灯片",
  "presentation", "slides", "keynote", "powerpoint"。
  Also trigger when users want to create visual storytelling content,
  even if they don't explicitly say "PPT" or "presentation".
  This skill creates HTML-based presentations with modern, dark-themed designs
  inspired by tech-decode and deep-dive content styles.
---

# PPT Writer - HTML演示文稿生成器

## 概述
帮助用户创建具有强烈视觉冲击力的HTML演示文稿。采用深色科技风格，适合技术解密、产品发布、深度分析等场景。

## 设计风格参考

### 参考文件
完整的样式参考和示例请查看：`references/演示文档.html`

### 视觉风格
- **主题**: 深色科技风（Dark Tech Theme）
- **配色方案**:
  - 背景: `#0a0a0f` (深黑蓝)
  - 主色调: `#00d4ff` (科技蓝)
  - 强调色: `#b44aff` (紫色)
  - 高亮色: `#ff4a8d` (粉红)
  - 成功色: `#00ff88` (绿色)
  - 警示色: `#ff8c00` (橙色)
  - 金色: `#ffd700` (用于评分/重要数据)
- **渐变**: `linear-gradient(135deg, #00d4ff, #b44aff, #ff4a8d)`

### 设计元素
1. **卡片系统**: 圆角16px，悬停效果，顶部渐变条
2. **数据展示**: 大号数字 + 小标签，网格布局
3. **引用块**: 左侧紫色边框，作者署名
4. **流程图**: 步骤卡片 + 箭头连接
5. **评分板**: 金色边框，大字体评分
6. **技能卡**: 图标 + 名称 + 进度条 + 评分

## 工作流程

### Step 1: 理解需求
询问用户以下信息：
1. **主题**: 演示文稿的核心主题是什么？
2. **目标受众**: 给谁看？（技术人员/管理层/普通用户）
3. **页数/结构**: 大概需要多少页？有什么必须包含的章节？
4. **风格偏好**: 是否需要调整配色或风格？
5. **关键信息**: 有哪些必须展示的数据、结论或观点？

### Step 2: 规划结构
根据主题设计演示文稿结构：

**标准结构模板**:
```
1. 封面页 (Hero)
   - 主标题（大字、渐变文字）
   - 副标题/简介
   - 关键数据卡片（3-5个核心数字）

2. 背景介绍 (Background)
   - 章节编号 (PART 01)
   - 大数字冲击区
   - 核心要点卡片

3. 主体内容 (Content Sections)
   - 每个PART包含：
     - 章节标题
     - 引用/观点
     - 详细内容（卡片/流程/对比）

4. 总结/结论 (Conclusion)
   - 核心发现
   - 评分/评价
   - 行动号召
```

### Step 3: 生成HTML
使用以下技术栈生成单文件HTML：

**必须包含的元素**:
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[演示标题]</title>
  <style>
    /* CSS变量定义 */
    :root {
      --bg: #0a0a0f;
      --bg2: #12121a;
      --card: #1a1a2e;
      --card2: #222240;
      --blue: #00d4ff;
      --purple: #b44aff;
      --pink: #ff4a8d;
      --green: #00ff88;
      --orange: #ff8c00;
      --yellow: #ffd700;
      --red: #ff4444;
      --text: #e8e8f0;
      --text2: #a0a0b8;
      --muted: #666680;
      --gm: linear-gradient(135deg, #00d4ff, #b44aff, #ff4a8d);
      --gf: linear-gradient(135deg, #ff4444, #ff8c00, #ffd700);
    }
    
    /* 核心样式 */
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { 
      font-family: 'Noto Sans SC', sans-serif; 
      background: var(--bg); 
      color: var(--text);
      line-height: 1.9;
    }
    
    /* 渐变文字 */
    .gt { 
      background: var(--gm); 
      -webkit-background-clip: text; 
      -webkit-text-fill-color: transparent; 
    }
    
    /* 卡片 */
    .card {
      background: var(--card);
      border: 1px solid rgba(180,74,255,0.12);
      border-radius: 16px;
      padding: 1.8rem;
      transition: 0.3s;
    }
    .card:hover {
      border-color: rgba(0,212,255,0.3);
      transform: translateY(-3px);
    }
    
    /* 数据展示 */
    .stat {
      text-align: center;
      padding: 1.2rem 1.8rem;
      background: rgba(26,26,46,0.6);
      border: 1px solid rgba(180,74,255,0.15);
      border-radius: 14px;
    }
    .stat-v {
      font-size: 2.2rem;
      font-weight: 900;
      background: var(--gm);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }
    
    /* 引用块 */
    .quote {
      border-left: 4px solid var(--purple);
      padding: 1.2rem 1.5rem;
      background: rgba(180,74,255,0.04);
      border-radius: 0 10px 10px 0;
    }
    
    /* 评分板 */
    .score-board {
      background: linear-gradient(135deg, rgba(26,26,46,0.9), rgba(34,34,64,0.9));
      border: 2px solid rgba(255,215,0,0.25);
      border-radius: 20px;
      padding: 2.5rem;
      text-align: center;
    }
    .score-big {
      font-size: 4rem;
      font-weight: 900;
      background: var(--gf);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }
  </style>
</head>
<body>
  <!-- 导航 -->
  <nav class="nav">...</nav>
  
  <!-- Hero区 -->
  <section class="hero">...</section>
  
  <!-- 内容区 -->
  <section class="s" id="section1">...</section>
  
  <!-- 更多章节... -->
</body>
</html>
```

### Step 4: 优化与交付
1. 检查响应式设计（移动端适配）
2. 确保所有渐变和动画效果正常
3. 提供文件保存建议

## 输出格式

生成完整的单文件HTML，包含：
- 内联CSS样式
- 响应式布局
- 悬停动画效果
- 滚动显示动画（可选）

## 设计组件库

### 1. 数据卡片 (Stat Card)
```html
<div class="stat">
  <div class="stat-v">数字</div>
  <div class="stat-l">标签</div>
</div>
```

### 2. 内容卡片 (Content Card)
```html
<div class="card" style="border-left:3px solid var(--blue)">
  <div class="card-title">标题</div>
  <div class="card-text">内容...</div>
  <span class="card-tag tag-b">标签</span>
</div>
```

### 3. 引用块 (Quote Block)
```html
<div class="quote">
  引用内容...
  <span class="author">-- 作者</span>
</div>
```

### 4. 流程图 (Flow Diagram)
```html
<div class="flow">
  <div class="flow-step">步骤1</div>
  <span class="flow-arr">→</span>
  <div class="flow-step">步骤2</div>
  ...
</div>
```

### 5. 评分板 (Score Board)
```html
<div class="score-board">
  <div class="score-big">9.4</div>
  <div class="score-label">综合评分</div>
</div>
```

### 6. 技能卡 (Skill Card)
```html
<div class="sk">
  <div class="sk-head">
    <div class="sk-icon">🔧</div>
    <div class="sk-score">9.5</div>
  </div>
  <div class="sk-name">技能名称</div>
  <div class="sk-tag">标签</div>
  <div class="sk-desc">描述...</div>
  <div class="sk-bar">
    <span class="sk-bar-label">维度</span>
    <div class="sk-bar-bg"><div class="sk-bar-fill" style="width:95%"></div></div>
    <span class="sk-bar-v">9.5</span>
  </div>
</div>
```

## 示例

**示例 1: 技术产品分析**
输入: "帮我做一个关于Claude Code源码分析的PPT"
输出: 生成深色科技风HTML演示文稿，包含：
- Hero区展示核心数据（文件数、代码行数等）
- 多章节结构（背景、架构、核心功能、评价）
- 卡片式内容展示
- 流程图展示技术架构
- 评分板总结

**示例 2: 项目汇报**
输入: "做一个季度工作总结的演示文稿"
输出: 生成商务风格HTML演示文稿，包含：
- 封面展示季度主题
- 关键指标数据卡片
- 项目进展时间线
- 成果展示
- 下季度计划

## 注意事项

1. **配色一致性**: 严格使用CSS变量定义的配色，确保视觉统一
2. **响应式设计**: 确保在移动设备上也能正常显示
3. **性能优化**: 使用CSS动画而非JS，减少性能开销
4. **可访问性**: 确保文字对比度足够，重要信息不依赖颜色 alone
5. **文件大小**: 尽量保持单文件，方便分享和演示
6. **浏览器兼容**: 使用现代CSS特性时添加必要的fallback

## 扩展能力

- 可以生成Reveal.js格式的幻灯片
- 可以导出为PDF（通过浏览器打印功能）
- 可以添加演讲者备注
- 可以集成图表库（如Chart.js）展示数据
