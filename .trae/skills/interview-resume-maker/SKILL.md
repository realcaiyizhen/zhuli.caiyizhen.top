---
name: interview-resume-maker
description: |
  帮助用户将Markdown格式的简历转换为高级感PDF简历。
  Use this skill when users want to convert their resume to PDF, create a printable resume, 
  generate a resume PDF, mention "简历转PDF", "简历打印", "简历PDF", "resume to PDF",
  or ask for "高级感简历", "设计感简历", "与众不同的简历".
  Also trigger when users need to format their resume for printing or want a visually appealing 
  resume presentation.
---

# Interview Resume Maker - 高级感简历制作工具

## 概述

帮助用户将Markdown格式的简历转换为**具有高级感设计的PDF文件**。

**核心目标**：生成的简历必须有**视觉冲击力**，与普通简历有**巨大区别**，同时**打印友好**。

**输出格式**：直接生成PDF文件，无需HTML中间步骤。

## 何时使用

- 用户有Markdown简历，想转成PDF
- 用户需要打印简历，想要A4格式的PDF
- 用户说"简历转PDF"、"简历打印"、"简历PDF"
- 用户要求"高级感"、"设计感"、"与众不同"的简历
- 用户想优化简历的展示效果或打印效果

## 设计规范（已固化）

### 字体系统（三层结构）

通过CSS变量统一管理，确保全站字体一致性：

```css
:root {
  --font-serif: 'Noto Serif SC', 'Georgia', serif;      /* 标题/姓名/公司名 */
  --font-sans: 'Noto Sans SC', 'Helvetica Neue', sans-serif;  /* 正文/描述 */
  --font-mono: 'JetBrains Mono', 'Consolas', monospace;  /* 标签/时间/代码 */
}
```

| 用途 | 字体 | 效果 |
|------|------|------|
| **标题/姓名/公司名** | `Noto Serif SC`（衬线体） | 高级感、权威感 |
| **正文/描述** | `Noto Sans SC`（无衬线） | 清晰易读 |
| **标签/时间/代码** | `JetBrains Mono`（等宽体） | 科技感、精确感 |

### 配色系统（黑白打印友好）

```css
:root {
  --primary: #000000;        /* 纯黑主色（打印清晰） */
  --accent: #333333;         /* 深灰强调（黑白打印呈深灰，有层次） */
  --accent-light: #f5f5f5;   /* 浅灰背景（黑白打印呈浅灰，不突兀） */
  --text: #1a1a1a;           /* 深灰文字 */
  --text-light: #4a4a4a;     /* 中灰次要文字 */
  --bg: #ffffff;             /* 纯白背景 */
  --card-bg: #ffffff;        /* 卡片纯白 */
  --border: #cccccc;         /* 中灰边框（黑白打印可见） */
}
```

**黑白打印设计原则**：
1. **不用彩色**：避免金色、蓝色等彩色，黑白打印会失真
2. **用灰度层次**：通过黑、深灰、中灰、浅灰创造层次
3. **避免深色背景**：不用深色背景块，黑白打印会变成黑块
4. **边框和线条**：用细线、边框创造视觉分隔，黑白打印清晰
5. **字体粗细**：用字重（font-weight）创造层次，而非颜色

### 项目配图规范

案例区域支持项目配图，增强真实感：

- **图片位置**：`temp/resume-files/project-pics/`
- **图片尺寸**：80×60px（打印版）/ 120×90px（如需要网页预览）
- **图片处理**：
  - 打印版：灰度滤镜 `filter: grayscale(30%)`，降低透明度 `opacity: 0.85`
  - 不追求清晰度，只追求"有图"的真实感
- **图片路径**：使用相对路径 `../temp/resume-files/project-pics/{图片名}.png`

## 工作流程

### Step 1: 读取简历源文件

读取用户指定的Markdown简历文件，提取完整内容。

**简历文件位置**：`temp/resume-files/`

### Step 2: 解析简历结构

**简历结构（四大块，项目放最后）**：
1. **自我介绍** - 个人简介 + 核心能力（4项）
2. **工作经历** - 按时间倒序排列，包含项目成果
3. **早期经历与教育** - 早期工作经历（3项）和教育背景
4. **项目介绍** - 8个重点项目的详细案例（放最后）

**为什么项目放最后？**
- 项目部分通常很长，不是每个面试官都有兴趣看
- 把项目放最后，前面三部分（自我介绍、工作经历、早期经历）构成一份**完整的简历**
- 面试官可以快速浏览核心信息，有兴趣再深入看项目详情
- 方便面试官按需阅读，提升简历的可读性

### Step 3: 生成PDF（核心）

使用Playwright或Puppeteer将HTML渲染为PDF：

```javascript
// PDF生成配置
const pdfOptions = {
  path: 'temp/{filename}.pdf',
  format: 'A4',
  printBackground: true,
  margin: {
    top: '15mm',
    right: '15mm',
    bottom: '15mm',
    left: '15mm'
  },
  preferCSSPageSize: true
};

// 等待字体加载完成后再生成PDF
await page.waitForFunction(() => {
  return document.fonts.ready;
});

await page.pdf(pdfOptions);
```

**关键要点**：
1. 等待Google Fonts加载完成后再生成PDF
2. 使用 `printBackground: true` 确保背景色打印
3. 设置A4纸张和15mm边距
4. 控制总页数在2-3页

### Step 4: 输出PDF文件

**输出位置**：`temp/`

**命名规范**：
- `temp/{原文件名}.pdf`
- 例如：`temp/resume-ai-product-manager.pdf`

## HTML模板（用于PDF渲染）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>姓名 - 职位</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@400;600;700&family=Noto+Sans+SC:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');

    :root {
      --primary: #000000;
      --accent: #333333;
      --accent-light: #f5f5f5;
      --text: #1a1a1a;
      --text-light: #4a4a4a;
      --bg: #ffffff;
      --card-bg: #ffffff;
      --border: #cccccc;
      --font-serif: 'Noto Serif SC', 'Georgia', serif;
      --font-sans: 'Noto Sans SC', 'Helvetica Neue', sans-serif;
      --font-mono: 'JetBrains Mono', 'Consolas', monospace;
    }

    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      font-family: var(--font-sans);
      background: var(--bg);
      color: var(--text);
      line-height: 1.7;
      font-size: 10.5pt;
    }

    .container {
      max-width: 210mm;
      margin: 0 auto;
      padding: 18mm 20mm;
      background: var(--card-bg);
    }

    /* 头部 - 名片级设计 */
    .header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding-bottom: 18px;
      border-bottom: 2.5px solid var(--primary);
      margin-bottom: 28px;
    }

    .header-left {
      display: flex;
      align-items: center;
      gap: 16px;
      flex: 1;
    }

    .header-avatar {
      width: 100px;
      height: 100px;
      border-radius: 50%;
      object-fit: cover;
      border: 2px solid var(--border);
      filter: grayscale(20%);
    }

    .header-name {
      font-family: var(--font-serif);
      font-size: 30pt;
      font-weight: 700;
      color: var(--primary);
      letter-spacing: 4px;
      margin-bottom: 6px;
    }

    .header-title {
      font-family: var(--font-serif);
      font-size: 13pt;
      color: var(--accent);
      font-weight: 600;
      letter-spacing: 2px;
    }

    .header-contact {
      text-align: right;
      font-size: 9pt;
      color: var(--text-light);
      line-height: 2;
    }

    .header-contact span {
      display: inline-block;
      padding: 0 8px;
      border-right: 1px solid var(--border);
    }

    .header-contact span:last-child { border-right: none; }

    /* 区块 */
    .section { margin-bottom: 24px; }

    .section-header {
      display: flex;
      align-items: baseline;
      margin-bottom: 14px;
      padding-bottom: 6px;
      border-bottom: 1px solid var(--border);
    }

    .section-label {
      font-family: var(--font-sans);
      font-size: 8.5pt;
      font-weight: 600;
      color: var(--text-light);
      letter-spacing: 2px;
      margin-right: 10px;
      text-transform: uppercase;
    }

    .section-title {
      font-family: var(--font-serif);
      font-size: 14pt;
      font-weight: 600;
      color: var(--primary);
    }

    /* 自我介绍 */
    .intro-text {
      font-size: 10pt;
      color: var(--text-light);
      line-height: 1.8;
      margin-bottom: 14px;
    }

    .intro-highlight {
      font-weight: 600;
      color: var(--primary);
    }

    .skills-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 12px 24px;
    }

    .skill-item {
      padding-left: 14px;
      border-left: 3px solid var(--accent);
    }

    .skill-name {
      font-family: var(--font-serif);
      font-weight: 700;
      color: var(--primary);
      font-size: 10pt;
      margin-bottom: 3px;
    }

    .skill-desc {
      font-size: 9pt;
      color: var(--text-light);
      line-height: 1.6;
    }

    /* 工作经历 */
    .job {
      margin-bottom: 20px;
      padding-left: 18px;
      border-left: 3px solid var(--accent);
      /* 工作经历允许跨页，避免大片空白 */
    }

    .job-header {
      page-break-after: avoid;
    }

    .job:last-child { margin-bottom: 0; }

    .job-header {
      display: flex;
      justify-content: space-between;
      align-items: baseline;
      margin-bottom: 4px;
      flex-wrap: wrap;
    }

    .job-title-row {
      display: flex;
      align-items: baseline;
      gap: 8px;
      flex-wrap: wrap;
    }

    .job-company {
      font-family: var(--font-serif);
      font-size: 11pt;
      font-weight: 600;
      color: var(--primary);
    }

    .job-role {
      font-size: 10pt;
      color: var(--accent);
      font-weight: 500;
    }

    .job-period {
      font-size: 8.5pt;
      color: var(--text-light);
      font-style: italic;
      white-space: nowrap;
    }

    .job-context {
      font-size: 9pt;
      color: var(--text-light);
      margin-bottom: 8px;
      line-height: 1.5;
      font-style: italic;
    }

    /* 项目卡片 */
    .project {
      background: var(--accent-light);
      padding: 10px 14px;
      margin-bottom: 8px;
      border-radius: 3px;
      page-break-inside: avoid;
    }

    .project:last-child { margin-bottom: 0; }

    .project-title {
      font-family: var(--font-serif);
      font-weight: 700;
      font-size: 9.5pt;
      color: var(--primary);
      margin-bottom: 4px;
    }

    .project-content {
      font-size: 9pt;
      color: var(--text-light);
      line-height: 1.6;
    }

    .project-content ul {
      margin: 0;
      padding-left: 14px;
    }

    .project-content li { margin: 2px 0; }

    .project-result {
      font-size: 9pt;
      color: var(--primary);
      font-weight: 700;
      margin-top: 6px;
      padding-top: 6px;
      border-top: 1px dashed var(--border);
    }

    /* 早期经历 */
    .early-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 12px 24px;
    }

    .early-item {
      padding-left: 14px;
      border-left: 3px solid var(--border);
    }

    .early-company {
      font-family: var(--font-serif);
      font-weight: 600;
      font-size: 10pt;
      color: var(--primary);
    }

    .early-role {
      font-size: 9pt;
      color: var(--text-light);
    }

    .early-period {
      font-size: 8.5pt;
      color: var(--text-light);
      font-style: italic;
    }

    .early-desc {
      font-size: 9pt;
      color: var(--text-light);
      line-height: 1.5;
      margin-top: 3px;
    }

    /* 教育背景 */
    .education {
      margin-top: 16px;
      padding: 10px 14px;
      background: var(--accent-light);
      border-radius: 3px;
    }

    .education-text {
      font-family: var(--font-serif);
      font-size: 10pt;
      color: var(--primary);
      font-weight: 600;
    }

    .education-sub {
      font-size: 9pt;
      color: var(--text-light);
    }

    /* 项目案例（带图片） */
    .case {
      background: var(--card-bg);
      border: 1px solid var(--border);
      padding: 14px 18px;
      margin-bottom: 12px;
      page-break-inside: avoid;
      display: grid;
      grid-template-columns: 80px 1fr 1fr;
      gap: 10px 16px;
    }

    .case:last-child { margin-bottom: 0; }

    .case-header {
      grid-column: 1 / -1;
      font-family: var(--font-serif);
      font-size: 11pt;
      font-weight: 600;
      color: var(--primary);
      padding-bottom: 6px;
      border-bottom: 1px solid var(--border);
    }

    .case-img {
      grid-row: 2 / 5;
      width: 80px;
      height: 60px;
      object-fit: cover;
      border-radius: 3px;
      filter: grayscale(30%) contrast(1.05);
      opacity: 0.85;
      align-self: start;
    }

    .case-row {
      font-size: 9pt;
      line-height: 1.6;
    }

    .case-label {
      font-weight: 700;
      color: var(--primary);
      display: inline;
    }

    .case-content {
      color: var(--text-light);
      display: inline;
    }

    .case-full {
      grid-column: 2 / -1;
    }

    .tag {
      display: inline-block;
      font-size: 8pt;
      padding: 1px 6px;
      border: 1px solid var(--border);
      border-radius: 2px;
      color: var(--text-light);
      margin-right: 4px;
      margin-bottom: 2px;
    }

    /* 打印优化 */
    @media print {
      @page {
        size: A4;
        margin: 15mm;
      }

      body {
        background: white;
        -webkit-print-color-adjust: exact !important;
        print-color-adjust: exact !important;
      }

      .container {
        padding: 0;
        max-width: 100%;
      }

      /* 打印分页控制策略：
         - 工作经历(.job)允许跨页，避免大片空白
         - 小块内容保持避免分页，确保可读性
         - 项目介绍强制分页，单独一页
      */
      .project, .case, .skill-item, .early-item {
        break-inside: avoid;
        page-break-inside: avoid;
      }

      .job-header {
        page-break-after: avoid;
      }

      .section-projects {
        page-break-before: always;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <!-- 头部 -->
    <div class="header">
      <div class="header-left">
        <img src="../temp/resume-files/avatar.png" alt="头像" class="header-avatar">
        <div>
          <div class="header-name">姓名</div>
          <div class="header-title">职位</div>
        </div>
      </div>
      <div class="header-contact">
        <span>📧 邮箱</span>
        <span>📱 电话</span>
        <span>📍 地点</span>
      </div>
    </div>

    <!-- 自我介绍 -->
    <div class="section">
      <div class="section-header">
        <span class="section-label">PROFILE</span>
        <span class="section-title">自我介绍</span>
      </div>
      <div class="intro-text">
        <span class="intro-highlight">核心经验</span>，个人简介...
      </div>
      <div class="skills-grid">
        <div class="skill-item">
          <div class="skill-name">能力名称</div>
          <div class="skill-desc">能力描述...</div>
        </div>
        <!-- 共4项 -->
      </div>
    </div>

    <!-- 工作经历 -->
    <div class="section">
      <div class="section-header">
        <span class="section-label">EXPERIENCE</span>
        <span class="section-title">工作经历</span>
      </div>
      <!-- 工作经历列表 -->
    </div>

    <!-- 早期经历与教育 -->
    <div class="section">
      <div class="section-header">
        <span class="section-label">EARLIER & EDUCATION</span>
        <span class="section-title">早期经历与教育</span>
      </div>
      <!-- 早期经历（3项）+ 教育背景 -->
    </div>

    <!-- 项目介绍 - 强制分页，单独一页 -->
    <div class="section section-projects">
      <div class="section-header">
        <span class="section-label">PROJECTS</span>
        <span class="section-title">项目介绍</span>
      </div>
      <!-- 项目案例（8项，带图片） -->
    </div>
  </div>
</body>
</html>
```

## 文件位置

简历文件统一存放在 `temp/resume-files/` 目录下（临时文件，不纳入Git管理）：

```
temp/resume-files/
├── resume-ai-product-manager.md      # AI产品经理简历
├── resume-ai-solution-expert.md      # AI解决方案专家简历
├── avatar.png                         # 头像
└── project-pics/                      # 项目案例配图
    ├── 第二大脑agent.png
    ├── 小红书创作agent.png
    ├── 专家证人助手.png
    ├── PBX客服助手.png
    ├── ai外呼.png
    ├── 爱盲购物商城和专利.png
    ├── 检索式对话机器人开发平台.png
    └── 互动小说.png
```

**读取规则**：
- 简历源文件从 `temp/resume-files/` 读取
- 项目图片从 `temp/resume-files/project-pics/` 读取
- 图片路径使用相对路径：`../temp/resume-files/project-pics/{图片名}.png`

## 输出文件命名规范

```
原文件名: resume-ai-product-manager.md
输出PDF: temp/resume-ai-product-manager.pdf
```

**输出位置**：所有生成的PDF文件统一输出到 `temp/` 目录下

## 注意事项

### 高级感设计要点（已固化到模板）

1. **字体**：三层字体系统（衬线体标题 + 无衬线正文 + 等宽标签）
2. **配色**：灰度层次（纯黑/深灰/中灰/浅灰），黑白打印友好
3. **层次**：通过边框、背景灰度、间距创造视觉层次
4. **留白**：适当的留白让简历呼吸
5. **配图**：项目案例配图，增强真实感

### PDF生成要点

1. **等待字体加载**：必须等待Google Fonts加载完成后再生成PDF
2. **打印背景**：使用 `printBackground: true` 确保背景色打印
3. **分页控制**：关键区块设置 `break-inside: avoid`
4. **页数控制**：控制在2-3页

### 内容组织要点

1. **自我介绍**：简洁有力，突出核心优势 + 4项核心能力
2. **工作经历**：倒序排列，重点突出最近3-5年，包含项目成果摘要
3. **早期经历与教育**：简化早期经历（3项），突出教育背景
4. **项目介绍**：8个重点项目，带配图，放最后方便按需阅读

## 示例

**示例 1: 生成PDF简历**
```
输入: "帮我把 resume.md 转成PDF简历"
输出: 生成A4格式的PDF文件，灰度层次设计、衬线体标题、专业排版、黑白打印友好
```

**示例 2: 更新简历设计**
```
输入: "更新简历，加入最新的项目"
输出: 重新生成PDF，使用固化的设计规范
```
