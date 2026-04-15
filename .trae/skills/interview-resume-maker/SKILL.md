---
name: interview-resume-maker
description: |
  帮助用户将Markdown格式的简历转换为HTML展示页面或A4打印版PDF简历。
  Use this skill when users want to convert their resume to HTML, create a printable resume, 
  generate a resume PDF, or mention "简历转HTML", "简历打印", "简历PDF", "resume to PDF".
  Also trigger when users need to format their resume for printing or want a visually appealing 
  resume presentation, even if they don't explicitly say "HTML" or "PDF".
---

# Interview Resume Maker - 简历制作工具

## 概述

帮助用户将Markdown格式的简历转换为两种格式的HTML文件：
1. **网页展示版** - 深色科技风，适合在线浏览
2. **A4打印版** - 白色背景、紧凑排版，适合打印成PDF

## 何时使用

- 用户有Markdown简历，想转成HTML展示页面
- 用户需要打印简历，想要A4格式的PDF
- 用户说"简历转HTML"、"简历打印"、"简历PDF"
- 用户想优化简历的展示效果或打印效果

## 工作流程

### Step 1: 理解需求

询问用户：
1. **目标格式**：网页展示版还是A4打印版？
2. **内容完整性**：是否需要删减内容，还是保持完整？
3. **特殊要求**：是否有特定的样式偏好？

### Step 2: 读取简历源文件

读取用户指定的Markdown简历文件，提取完整内容。

### Step 3: 选择模板并生成

根据需求选择对应模板：

#### 模板A：网页展示版（深色科技风）
- 深色背景 (#0a0a0f)
- 渐变文字效果
- 卡片式布局
- 悬停动画效果
- 适合在线浏览

#### 模板B：A4打印版（白色背景）
- 白色背景、黑色文字
- 紧凑排版（字体10.5pt，行高1.6）
- 两列布局充分利用空间
- 分页控制（break-inside: avoid）
- 适合打印成PDF

### Step 4: 输出文件

生成单文件HTML，包含内联CSS样式。

## 设计规范

### 网页展示版规范

```css
/* 配色 */
--bg: #0a0a0f;           /* 深黑蓝背景 */
--text: #e8e8f0;         /* 浅色文字 */
--accent: #00d4ff;       /* 科技蓝强调 */
--success: #00ff88;      /* 绿色成果 */

/* 字体 */
font-size: 16px (正文)
font-size: 14px (次要)

/* 布局 */
max-width: 1000px
padding: 2rem
卡片圆角: 16px
```

### A4打印版规范

```css
/* 配色 */
--bg: #ffffff;           /* 白色背景 */
--text: #1a1a1a;         /* 纯黑文字 */
--accent: #0066cc;       /* 蓝色强调 */
--success: #2e7d32;      /* 绿色成果 */

/* 字体 */
font-size: 10.5pt (正文)
font-size: 9pt (项目内容)

/* 布局 */
max-width: 210mm (A4宽度)
padding: 15mm
margin: 12mm (打印边距)

/* 分页 */
page-break-inside: avoid  /* 避免内容被分页切断 */
```

## 内容结构模板

### 头部区域
```html
<div class="header">
  <h1 class="header-name">姓名</h1>
  <div class="header-title">职位</div>
  <div class="header-contact">
    <span>📧 邮箱</span>
    <span>📱 电话</span>
    <span>📍 地点</span>
  </div>
</div>
```

### 工作经历区域
```html
<div class="job">
  <div class="job-header">
    <span class="job-title">职位</span>
    <span class="job-company"> · 公司</span>
    <span class="job-period">时间</span>
  </div>
  <p class="job-desc">职责描述</p>
  <div class="project">
    <div class="project-title">项目名称</div>
    <div class="project-content">
      <ul><li>工作内容</li></ul>
      <div class="project-result"><strong>成果：</strong>具体成果</div>
    </div>
  </div>
</div>
```

### 项目案例区域
```html
<div class="case">
  <div class="case-header">案例标题</div>
  <div class="case-row">
    <span class="case-label">背景：</span>
    <span class="case-content">背景描述</span>
  </div>
  <!-- 挑战、解决方案、成果 -->
</div>
```

## 打印设置指南

生成A4打印版后，按以下步骤打印为PDF：

1. **Chrome/Edge打开HTML文件**
2. **按 `Ctrl+P` 打开打印对话框**
3. **目标打印机选择"另存为PDF"**
4. **勾选"更多设置" → "背景图形"**
5. **纸张尺寸选择"A4"**
6. **边距选择"默认"或"无"**
7. **点击保存**

## 注意事项

1. **字体颜色**
   - 网页版：使用渐变和亮色，确保对比度
   - 打印版：使用纯黑 `#1a1a1a`，确保打印清晰

2. **分页控制**
   - 工作经历、项目、案例都设置 `break-inside: avoid`
   - 避免一个项目被分到两页

3. **内容精简**
   - A4打印版可能需要精简内容
   - 优先保留：职位、公司、时间、核心项目、量化成果

4. **响应式**
   - 网页版需要适配移动端
   - 打印版固定A4尺寸

## 示例

**示例 1: A4打印版简历**
```
输入: "帮我把 resume.md 转成A4打印版"
输出: 生成A4格式的HTML文件，白色背景、紧凑排版、适合打印
```

**示例 2: 网页展示版简历**
```
输入: "把简历转成HTML展示页面"
输出: 生成深色科技风的HTML文件，适合在线浏览
```

**示例 3: 优化现有简历**
```
输入: "这个简历打印出来字体看不清，帮我优化"
输出: 分析原文件问题，重新生成打印友好的版本
```

## 知识库使用

根据用户指定的简历文件读取内容。实际关联的知识库文件列表见 AGENTS.md 中的 Skill 与知识库关系表。

## 输出文件命名规范

```
原文件名: resume-ai-product-manager.md
网页版: resume-ai-product-manager-web.html
打印版: resume-ai-product-manager-print.html
```
