---
name: skill-creator-enhanced
description: |
  创建和优化 Trae Skill 的增强版工具，融合 Anthropic 最佳实践。
  ALWAYS use this skill when users want to create a new skill, write a SKILL.md file, 
  improve an existing skill, or need guidance on skill development.
  This enhanced version includes knowledge base integration, progressive disclosure patterns, 
  and scientific evaluation methods.
  当用户提到创建 skill、编写 SKILL.md、优化现有 skill、skill 开发指南时，优先使用此 skill。
---

# Skill Creator Enhanced

一个融合 Anthropic 最佳实践和 Trae 原生方法的 Skill 开发工具。

## 核心理念

Skill 是教 Claude 如何完成特定任务的指南。好的 Skill 应该：
- **精准触发**：在正确的时候被调用
- **清晰指令**：让 Claude 知道具体怎么做
- **渐进披露**：分层加载，节省上下文
- **可测试**：能够验证效果

---

## 创建新 Skill 的完整流程

### Step 1: 捕获意图（Capture Intent）

在写代码之前，先回答这 4 个问题：

1. **这个 Skill 要解决什么问题？**
   - 具体场景是什么？
   - 用户会怎么描述这个需求？

2. **什么时候应该触发？**
   - 用户说什么关键词时触发？
   - 有哪些同义词或相关表达？

3. **输出格式是什么？**
   - 文本回复？
   - 文件生成？
   - 代码？

4. **是否需要测试？**
   - 客观可验证的输出（文件处理、数据提取）→ 建议测试
   - 主观输出（写作风格、创意）→ 可不测试

### Step 2: 检查知识库（可选）

如果 Skill 需要共享的背景信息，可以创建知识库文件：

```
knowledge/
├── career-strategy.md      # 职业发展战略
├── family-system.md        # 家庭系统分析
└── ...
```

**设计原则**：
- 多个 Skill 需要共享的背景信息 → 放 `knowledge/`
- Skill 私有的详细文档 → 放 Skill 自己的 `references/`
- 在 Skill 中显式指定读取哪个知识库文件

**知识库文件格式建议**：
```markdown
---
title: 知识库标题
description: 内容摘要
tags: [标签1, 标签2]
related_skills: [skill1, skill2]
---

# 内容...
```

### Step 3: 设计目录结构

```
skill-name/
├── SKILL.md                    # 核心文件（必须）
│   ├── YAML frontmatter        # name, description
│   └── Markdown 指令            # 分层组织的指令
├── scripts/                    # 可执行脚本（可选）
│   └── helper.py
├── references/                 # 参考文档（可选）
│   └── advanced-guide.md
└── assets/                     # 资源文件（可选）
    └── template.docx
```

### Step 4: 编写 SKILL.md

#### 4.1 YAML Frontmatter

```yaml
---
name: skill-name                    # 小写，用-连接
description: |
  描述这个 skill 做什么，以及什么时候使用。
  Use this skill when users ask about X, Y, or Z.
  Also trigger when users mention related concepts like A, B, C.
  确保在 description 中包含多种触发场景，因为 Claude 倾向于 undertrigger。
---
```

**description 写作要点**：
- ✅ 明确说 "Use this skill when..."
- ✅ 列举 3+ 种触发场景（包括同义词）
- ✅ 加一句 "even if they don't explicitly ask for X"
- ✅ 语气主动（pushy），不要太保守
- ❌ 不要只写 "How to do X"，不写触发条件

#### 4.2 Markdown Body 结构

```markdown
# Skill Name

## 概述
简要说明这个 skill 的用途。

## 何时使用
- 场景 1: ...
- 场景 2: ...
- 场景 3: ...

## 工作流程
### Step 1: ...
### Step 2: ...
### Step 3: ...

## 输出格式
明确指定输出格式：
```
模板...
```

## 示例
**示例 1:**
输入: ...
输出: ...

**示例 2:**
输入: ...
输出: ...

## 关联知识库
- `knowledge/xxx.md` - 包含背景信息
- `knowledge/yyy.md` - 包含案例数据

## 注意事项
- 注意点 1
- 注意点 2
```

**渐进式披露原则**：
1. **Level 1**: YAML frontmatter（始终可见，用于触发判断）
2. **Level 2**: SKILL.md body（触发后加载，<500行）
3. **Level 3**: references/, scripts/（按需加载）

#### 4.3 References 目录使用指南

`references/` 目录用于存放 Skill 私有的详细参考文档，当 SKILL.md 内容过长或需要模块化组织时使用。

**何时使用 references：**
- 详细的技术规范或API文档
- 复杂的示例集合
- 长篇的最佳实践指南
- 需要按需加载的辅助内容

**目录结构示例：**
```
skill-name/
├── SKILL.md
└── references/
    ├── api-reference.md      # API详细说明
    ├── examples.md           # 更多示例
    └── advanced-patterns.md  # 高级用法
```

**在 SKILL.md 中引用 references：**
```markdown
## 参考资料

根据问题复杂度，查阅以下文档：
- `references/api-reference.md` - 完整的API参数说明
- `references/examples.md` - 更多使用示例

> 当用户询问具体参数或高级用法时，读取对应的 reference 文件
```

**使用原则：**
1. **按需加载**：不要在 Skill 触发时自动读取所有 references
2. **显式引用**：在指令中明确说明何时读取哪个 reference 文件
3. **保持独立**：每个 reference 文件应该是相对独立的主题
4. **路径正确**：使用相对路径 `references/filename.md`

### Step 5: 测试 Skill

#### 5.1 准备测试用例

准备 3 个真实的用户问题：

```markdown
## 测试用例

### 测试 1: 应该触发
问题: "..."
预期: Skill 被触发，输出符合预期

### 测试 2: 应该触发（不同表达方式）
问题: "..."
预期: Skill 被触发，输出符合预期

### 测试 3: 不应该触发
问题: "..."
预期: Skill 不被触发（避免误触发）
```

#### 5.2 执行测试

**测试触发准确性**：
1. 用测试问题问 Claude
2. 观察是否触发了目标 Skill
3. 记录触发率

**测试输出质量**：
1. 检查输出是否符合预期格式
2. 检查是否遵循了指令
3. 检查是否使用了关联的知识库

#### 5.3 记录结果

```markdown
## 测试结果记录

| 测试问题 | 是否触发 | 输出质量 | 备注 |
|---------|---------|---------|------|
| 问题1 | ✅/❌ | 好/中/差 | ... |
| 问题2 | ✅/❌ | 好/中/差 | ... |
| 问题3 | ✅/❌ | 好/中/差 | ... |
```

---

## 优化现有 Skill

### Description 优化检查清单

- [ ] 是否明确说了 "Use this skill when..."？
- [ ] 是否列举了 3 种以上的触发场景？
- [ ] 是否包含了同义词和相关概念？
- [ ] 是否强调了 "even if they don't explicitly ask for X"？
- [ ] 长度是否控制在 100 词左右？
- [ ] 语气是否主动（pushy）而不是保守？

### 结构优化检查清单

- [ ] SKILL.md 是否 <500 行？
- [ ] 大段内容（>300行）是否移到了 references/？
- [ ] 重复逻辑是否抽成了 scripts/？
- [ ] 是否有关联知识库的说明？
- [ ] 指令是否解释了 "why" 而不是只用 MUST？

### 迭代优化流程

```
测试 → 发现问题 → 修改 Skill → 再测试 → 对比改进
```

**关键原则**：
1. **从反馈中泛化**：不要只为测试用例做特殊处理
2. **保持简洁**：删除不拉重量的内容
3. **解释原因**：让 Claude 理解为什么要这样做
4. **相信 LLM**：给框架，让 Claude 自己发挥

---

## Skill 写作最佳实践

### 1. 使用祈使句

```markdown
✅ 推荐:
检查 knowledge/ 目录
询问用户具体需求
生成报告

❌ 不推荐:
你应该检查 knowledge 目录
你可以询问用户
我们可以生成报告
```

### 2. 明确输出格式

```markdown
✅ 推荐:
## 报告结构
ALWAYS use this exact template:
# [Title]
## Executive summary
## Key findings
## Recommendations

❌ 不推荐:
生成一个报告，包含标题、摘要、发现和建议
```

### 3. 使用示例

```markdown
## 提交信息格式

**示例 1:**
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication

**示例 2:**
Input: Fixed login bug on mobile
Output: fix(login): resolve mobile authentication issue
```

### 4. 知识库联动（可选）

如果你的项目使用 `knowledge/` 目录共享背景信息：

```markdown
## 关联知识库

根据问题类型，查阅以下知识库文件：
- `knowledge/career-strategy.md` - 职业发展战略
- `knowledge/resume&projects.md` - 工作经历和项目

> 在 Skill 指令中显式指定读取哪个文件，例如：
> "如果用户问职业规划，先读取 `knowledge/career-strategy.md`"
```

**知识库 vs Skill 私有文档**：

| 类型 | 位置 | 用途 |
|------|------|------|
| 共享知识库 | `knowledge/` | 多个 Skill 共用的背景信息 |
| Skill 私有文档 | `skill-name/references/` | 该 Skill 专用的详细指南 |

**示例**：
```markdown
## 参考资料

- `knowledge/family-system.md` - 家庭系统背景（共享知识库）
- `references/communication-patterns.md` - 沟通技巧详解（Skill 私有）
```
```

---

## 与 Trae 内置 skill-creator 的协作

这个 enhanced 版本**补充而非替代** Trae 内置的 skill-creator：

| 场景 | 使用哪个 |
|------|---------|
| 快速创建简单 Skill | 内置 skill-creator |
| 需要知识库联动 | 本 enhanced 版本 |
| 需要科学评估流程 | 本 enhanced 版本 |
| 优化现有 Skill | 本 enhanced 版本 |

**兼容策略**：
- 本 Skill 的 description 明确说 "ALWAYS use this skill when..."
- 在指令中保留了 Trae 原生方法的核心逻辑
- 添加了 Anthropic 的最佳实践作为增强

---

## 示例：创建一个完整的 Skill

### 需求
创建一个帮助用户写周报的技能。

### 执行过程

**Step 1: 捕获意图**
- 问题：帮助用户快速生成周报
- 触发：用户说"写周报"、"weekly report"、"总结这周工作"
- 输出：Markdown 格式的周报
- 测试：是，可以验证格式

**Step 2: 检查知识库**
- 发现 `knowledge/work-notes.md` 包含工作记录模板
- 决定关联此知识库

**Step 3: 创建目录**
```
weekly-report/
├── SKILL.md
└── assets/
    └── template.md
```

**Step 4: 编写 SKILL.md**
```yaml
---
name: weekly-report
description: |
  帮助用户快速生成周报。
  Use this skill when users ask to write a weekly report, 
  summarize weekly work, create work summaries, or mention 
  "周报", "weekly report", "这周工作总结".
  Also trigger when users want to document their weekly achievements,
  even if they don't explicitly say "weekly report".
---

# Weekly Report Generator

## 概述
帮助用户快速生成结构化的周报。

## 工作流程

### Step 1: 收集信息
询问用户：
1. 这周主要完成了什么工作？
2. 遇到了什么挑战？
3. 下周计划做什么？

### Step 2: 生成周报
使用以下模板生成周报：

```markdown
# 周报 - [日期范围]

## 本周完成
- [工作项1]
- [工作项2]

## 遇到的问题
- [问题1] - [解决方案]

## 下周计划
- [计划1]
- [计划2]

## 其他备注
[其他需要记录的内容]
```

## 参考资料（可选）
- 如果有共享知识库: `knowledge/work-notes.md` - 工作记录模板
- 如果 Skill 私有: `references/weekly-template.md` - 周报模板

## 示例

**示例 1:**
输入: "帮我写个周报，这周完成了项目A的需求评审和原型设计"
输出: [生成的周报]
```

**Step 5: 测试**
- 测试1: "帮我写周报" → ✅ 触发，输出正确
- 测试2: "总结这周工作" → ✅ 触发，输出正确
- 测试3: "帮我写代码" → ✅ 不触发（避免误触发）

---

## 总结

创建好的 Skill 需要：
1. **清晰的需求理解** - 知道要解决什么问题
2. **精准的 description** - 让 Claude 在正确时候触发
3. **分层的指令结构** - 渐进式披露，节省上下文
4. **科学的测试验证** - 确保 Skill 有效
5. **持续的迭代优化** - 根据反馈改进

记住：**Skill 是教 Claude 做事的指南，不是死记硬背的脚本**。解释 "why"，给好框架，让 Claude 发挥智能。
