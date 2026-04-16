---
title: "专家证人资料检索助手 - 产品需求文档(PRD)"
version: "1.0"
status: "待开发"
author: "蔡大"
date: "2026-04-16"
tags: ["PRD", "专家证人", "法律科技", "AI检索", "PDF处理"]
---

# 专家证人资料检索助手 - PRD

## 1. 项目概述

### 1.1 产品定位
面向法律诉讼场景的专家证人辅助工具，解决海量证据材料（PDF）的快速检索与关联分析问题。

### 1.2 核心价值
- **效率提升**：从人工翻阅千页PDF到秒级智能检索
- **精准定位**：基于语义相似度，而非关键词匹配
- **证据关联**：自动发现与争议焦点相关的证据材料

### 1.3 目标用户
- 专家证人（技术/行业专家）
- 代理律师
- 企业法务

---

## 2. 功能需求

### 2.1 核心功能

#### 功能1：PDF批量导入与处理
| 需求项 | 描述 |
|--------|------|
| 输入 | 批量PDF文件（支持拖拽/文件夹选择） |
| 处理流程 | PDF → 图片转换 → 向量化 → 本地存储 |
| 输出 | 建立可检索的向量索引库 |
| 技术方案 | Apache PDFBox（DPI=150，PNG格式） |

#### 功能2：语义检索
| 需求项 | 描述 |
|--------|------|
| 输入 | 自然语言查询（如"合同签署日期相关的证据"） |
| 处理 | 查询向量化 → 相似度计算 → Top-N返回 |
| 输出 | 相关PDF页面列表（含相似度分数） |
| 技术方案 | 阿里云百炼多模态嵌入 + SQLite-vec本地检索 |

#### 功能3：争议焦点管理
| 需求项 | 描述 |
|--------|------|
| 功能 | 创建/编辑争议焦点清单 |
| 关联 | 每个焦点可关联检索到的证据 |
| 导出 | 生成争议-证据对照表 |

#### 功能4：证据标注与笔记
| 需求项 | 描述 |
|--------|------|
| 功能 | 对检索结果添加标注和笔记 |
| 持久化 | 标注数据本地存储 |
| 导出 | 支持导出带标注的报告 |

### 2.2 功能优先级

```
P0（MVP核心）
├── PDF批量导入
├── 语义检索（文本查询）
└── 结果展示（PDF预览+页码定位）

P1（增强体验）
├── 争议焦点管理
├── 证据标注
├── 检索历史
└── 批量导出

P2（高级功能）
├── 图片检索（以图搜图）
├── 双方观点对比
└── 报告生成
```

---

## 3. 技术架构

### 3.1 技术栈选型

| 层级 | 技术选型 | 理由 |
|------|---------|------|
| 前端 | JavaFX / Electron | 跨平台桌面应用 |
| 后端 | Java 17+ | 主力开发语言 |
| PDF处理 | Apache PDFBox 3.0.7 | 开源免费，成熟稳定 |
| 向量化 | 阿里云百炼 API | 成本低（千页约0.23元） |
| 向量存储 | SQLite + sqlite-vec | 单文件，零依赖 |
| 数据库 | SQLite | 元数据存储 |

### 3.2 数据流架构

```
┌─────────────────────────────────────────────────────────┐
│                      用户界面层                          │
│  PDF导入 │ 检索输入 │ 结果展示 │ 焦点管理 │ 标注功能     │
├─────────────────────────────────────────────────────────┤
│                      业务逻辑层                          │
│  导入服务 │ 检索服务 │ 焦点服务 │ 标注服务 │ 导出服务     │
├─────────────────────────────────────────────────────────┤
│                      数据处理层                          │
│  PDFBox转换 │ 百炼API向量化 │ 相似度计算 │ 结果排序      │
├─────────────────────────────────────────────────────────┤
│                      数据存储层                          │
│  SQLite-vec（向量） │ SQLite（元数据/标注） │ 文件系统    │
└─────────────────────────────────────────────────────────┘
```

### 3.3 核心数据结构

#### 文档表 (documents)
```sql
CREATE TABLE documents (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    file_path TEXT NOT NULL,           -- PDF文件路径
    file_name TEXT NOT NULL,           -- 文件名
    total_pages INTEGER,               -- 总页数
    case_id TEXT,                      -- 案件编号
    import_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status TEXT DEFAULT 'active'       -- active/deleted
);
```

#### 页面向量表 (page_vectors)
```sql
CREATE TABLE page_vectors (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    doc_id INTEGER NOT NULL,
    page_number INTEGER NOT NULL,      -- 页码（从1开始）
    image_path TEXT,                   -- 页面图片路径
    embedding BLOB,                    -- 向量数据（维度1536）
    FOREIGN KEY (doc_id) REFERENCES documents(id)
);

-- 创建向量索引（sqlite-vec扩展）
CREATE VIRTUAL TABLE vec_index USING vec_index(
    embedding FLOAT[1536]
);
```

#### 争议焦点表 (focus_points)
```sql
CREATE TABLE focus_points (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    case_id TEXT NOT NULL,
    title TEXT NOT NULL,               -- 焦点标题
    description TEXT,                  -- 详细描述
    created_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 证据关联表 (evidence_links)
```sql
CREATE TABLE evidence_links (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    focus_id INTEGER NOT NULL,
    page_vector_id INTEGER NOT NULL,
    relevance_score REAL,              -- 关联度分数
    notes TEXT,                        -- 备注
    created_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 4. 接口设计

### 4.1 核心服务接口

#### PdfImportService
```java
public interface PdfImportService {
    /**
     * 批量导入PDF文件
     * @param pdfFiles PDF文件列表
     * @param caseId 案件编号
     * @return 导入任务ID
     */
    String batchImport(List<File> pdfFiles, String caseId);
    
    /**
     * 获取导入进度
     * @param taskId 任务ID
     * @return 进度信息（0-100）
     */
    ImportProgress getProgress(String taskId);
}
```

#### VectorSearchService
```java
public interface VectorSearchService {
    /**
     * 语义检索
     * @param query 查询文本
     * @param caseId 案件编号（限定检索范围）
     * @param topK 返回结果数量
     * @return 检索结果列表
     */
    List<SearchResult> search(String query, String caseId, int topK);
    
    /**
     * 相似页面检索（以图搜图）
     * @param pageVectorId 页面ID
     * @param topK 返回结果数量
     * @return 相似页面列表
     */
    List<SearchResult> searchSimilar(long pageVectorId, int topK);
}

public class SearchResult {
    private long pageVectorId;      // 页面向量ID
    private long docId;             // 文档ID
    private String fileName;        // 文件名
    private int pageNumber;         // 页码
    private String imagePath;       // 页面图片路径
    private float similarityScore;  // 相似度分数（0-1）
}
```

#### FocusPointService
```java
public interface FocusPointService {
    /**
     * 创建争议焦点
     */
    long createFocusPoint(String caseId, String title, String description);
    
    /**
     * 关联证据
     */
    void linkEvidence(long focusId, long pageVectorId, float score, String notes);
    
    /**
     * 获取焦点的证据列表
     */
    List<EvidenceLink> getEvidenceLinks(long focusId);
    
    /**
     * 导出争议-证据对照表
     */
    File exportReport(String caseId, String outputPath);
}
```

---

## 5. 非功能需求

### 5.1 性能指标

| 指标 | 目标值 | 说明 |
|------|--------|------|
| PDF导入速度 | 10页/秒 | 含转换+向量化 |
| 检索响应时间 | <2秒 | 百万级数据 |
| 并发导入 | 支持 | 多线程处理 |
| 存储占用 | <2MB/页 | 图片+向量+元数据 |

### 5.2 可靠性

- **数据安全**：所有数据本地存储，不上传云端（除向量化API调用）
- **备份机制**：SQLite单文件，支持随时复制备份
- **异常恢复**：导入任务断点续传

### 5.3 用户体验

- **进度可见**：批量导入显示实时进度
- **预览功能**：检索结果可直接预览PDF页面
- **快捷键支持**：常用操作支持快捷键

---

## 6. 开发计划

### 6.1 里程碑规划

| 阶段 | 周期 | 交付物 | 验收标准 |
|------|------|--------|---------|
| **MVP** | 2周 | 核心检索功能 | 支持PDF导入+语义检索+结果展示 |
| **V1.0** | 4周 | 完整功能 | 含焦点管理+标注+导出 |
| **V1.1** | 2周 | 优化迭代 | 性能优化+用户体验改进 |

### 6.2 MVP任务拆解

**Week 1**
- [ ] 项目脚手架搭建（JavaFX界面框架）
- [ ] SQLite数据库设计+初始化
- [ ] PDFBox集成（PDF转图片）
- [ ] 阿里云百炼API封装

**Week 2**
- [ ] 向量存储（sqlite-vec集成）
- [ ] 语义检索功能
- [ ] 结果展示界面
- [ ] 端到端测试

### 6.3 成本估算

**开发成本（单人Vibe Coding）**
```
MVP阶段：2周 = 10人天
V1.0阶段：4周 = 20人天
总计：30人天 ≈ 1.5个月
```

**运行成本**
```
向量化成本：1000页 × 0.00023元/页 = 0.23元
存储成本：本地存储，无额外费用
```

---

## 7. 风险与应对

| 风险 | 影响 | 应对措施 |
|------|------|---------|
| 阿里云API调用失败 | 高 | 实现重试机制+本地缓存 |
| 大PDF文件内存溢出 | 中 | 分页处理+流式读取 |
| 向量检索精度不足 | 中 | 调优DPI+支持人工筛选 |
| 用户数据丢失 | 高 | 自动备份+导出功能 |

---

## 8. 附录

### 8.1 技术参考
- [Apache PDFBox文档](https://pdfbox.apache.org/)
- [阿里云百炼多模态嵌入API](https://help.aliyun.com/zh/model-studio/developer-reference/multimodal-embedding-api-reference)
- [sqlite-vec项目](https://github.com/asg017/sqlite-vec)

### 8.2 相关文件
- [专家证人助手项目规划](./expert-witness-assistant.md)

---

## 更新记录

| 日期 | 版本 | 变更内容 |
|------|------|---------|
| 2026-04-16 | 1.0 | 初始版本，确定MVP功能与技术方案 |
