# 空间转录组入门：NCI 给癌症研究者的「零基础路线图」

> 解读笔记 | 美国国家癌症研究所（NCI）CBIIT 培训指南  
> 原文链接：[Spatial Transcriptomics - NCI](https://www.cancer.gov/about-nci/organization/cbiit/training/library/st)  
> 更新日期：2025-09-03

---

## 摘要

Bulk RNA-seq 看的是组织「平均表达」，单细胞 RNA-seq 能分辨细胞类型，却丢掉了**空间位置**。

在肿瘤研究里，这往往意味着：你不知道哪些基因在肿瘤内活跃、哪些在邻近免疫细胞里活跃，更说不清**距离肿瘤远近**如何影响免疫细胞的表达状态。

**空间转录组（Spatial Transcriptomics, ST）** 正是为了补上这块拼图。NCI 的这份培训指南面向数据科学家和癌症研究者，系统梳理了 ST 是什么、为什么重要、怎么选型，以及如何用 **spatialGE** 零代码上手。

---

## 一、ST 是什么？比 bulk / 单细胞多看到了什么？

ST 能在组织原位**绘制基因表达的空间分布图**，让你看到表达模式如何随区域变化——不仅知道「表达了什么」，还知道「在哪里表达」。

借助 ST，你可以：

- 识别**肿瘤及邻近细胞**中哪些基因处于活跃状态
- 比较**靠近肿瘤 vs 远离肿瘤**的免疫细胞基因表达差异
- 基于**细胞间相互作用**发现新的治疗靶点
- 理解细胞如何通讯、适应或**抵抗治疗**
- 获得比 bulk / 单细胞 RNA-seq 更完整的**肿瘤微环境（TME）**视图

> **一句话：ST = 基因表达快照 + 组织架构上下文。**

---

## 二、为什么对癌症研究特别重要？

### 2.1 验证科学假说

你可能对某种细胞或 TME 机制已有假说，ST 能帮你「填拼图」——在完整组织微环境背景下观察基因表达，进一步 refine 假说、提出新问题，再用 scRNA-seq 或功能实验验证。

### 2.2 走向临床转化

ST 结果不局限于基础研究：

| 应用方向 | 具体价值 |
|----------|----------|
| **药物靶点发现** | 识别潜在分子靶点 |
| **候选药物评估** | 观察药物在组织中的作用模式 |
| **精准治疗** | 筛选最可能响应特定药物的患者 |
| **耐药监测** | 检测治疗抵抗相关的空间表达特征 |

---

## 三、动手之前：先回答 4 个问题

NCI 强调：**先定义研究问题，再选技术和工具。**

开工前问自己：

1. **组织类型是什么？** — FF（新鲜冷冻）还是 FFPE（石蜡包埋）？
2. **样本规模多大？** — 捕获面积、每片细胞数等
3. **需要多高的分辨率？** — 精细到细胞类型/微环境，还是区域级趋势即可？
4. **有没有高质量 scRNA-seq 参考？** — 是否包含目标细胞群，用于去卷积/注释？

这四个问题，直接决定你该走「成像路线」还是「测序路线」。

---

## 四、两大技术路线：怎么选？

### 4.1 成像法 ST（荧光探针 / 原位杂交）

**适合：** 小范围、高精度、需要亚细胞细节

- 可将表达映射到**细胞边界、细胞器**（如细胞核、胞外区域）
- 看清**邻近细胞如何相互作用**
- 通常采用**靶向 panel**（数百至数千基因），panel 需针对科学问题定制
- 代表平台：**MERFISH、seqFISH、CosMx、Xenium**

### 4.2 测序法 ST（空间条形码阵列 + NGS）

**适合：** 大样本、全组织切片、区域/域级分析

- 适合整片肿瘤等**大面积样本**
- 可见目标基因在组织中的**空间位置**，再通过 NGS 定量
- 分辨率从 ROI、spot 到近细胞级不等
- 分辨率越高，**dropout（零值）** 往往越严重，低层级细胞亚群越难分辨
- 代表平台：**10x Visium、Slide-seq** 等

### 4.3 NCI 的一句话建议

> **小区域要细节 → 成像法；大区域看域/生态位 → 测序法。**  
> 必要时**两种方法组合**，弥补单一技术难以捕捉全部细胞与互作的局限。

---

## 五、分析工具：从商业平台到 spatialGE

商业平台之外，NCI 重点推荐了 **spatialGE**——NCI 资助的**开源、点选式**分析平台，特别适合生物信息学新手。

### 5.1 spatialGE 能做什么？

| 功能模块 | 说明 |
|----------|------|
| 数据导入与整理 | 支持 10x Visium、CosMx 等商业平台数据，也支持通用格式 |
| QC 与数据转换 | 质控、归一化 |
| 可视化 | PCA、UMAP、热图 + 空间背景 |
| 空间域检测 | 聚类识别组织区域 |
| 细胞表型/去卷积 | 整合 SpaGCN、STdeconvolve、InSituType |
| 空间统计 | Hotspot 检测、表达梯度分析 |

**亮点：**

- 无需编程，也能调用对癌症研究常用的高级算法
- 结果可导出表格和高清图
- 分析参数自动记录，便于**复现**
- 提供 Web 版与 R 包版本

### 5.2 使用 spatialGE 的三条实用提示

1. **按步骤来** — 模块化设计，完成「数据导入 → QC → 可视化 → …」才会解锁下一步，避免跳过关键质控
2. **复杂分析需耐心** — 空间可视化很快；空间基因集富集等高级统计随样本量/spot 数增加而变慢，可设邮件通知
3. **注意数据合规** — 虽有账户密码保护，**不要上传 PHI（受保护健康信息）**，遵守 HIPAA

---

## 六、spatialGE 分析流程一览

```
导入数据（Visium / CosMx / 自定义 count + 坐标 + 图像）
        ↓
QC 与数据转换
        ↓
可视化（PCA / UMAP / 热图 + 空间图）
        ↓
空间域检测 / 去卷积 / Hotspot / 梯度分析
        ↓
导出结果（表格 + 图片 + 参数文件 → 可复现）
```

### 典型探索方向

- 哪些基因在**特定空间区域**高表达？
- 哪些基因集呈现 **hotspot** 模式？
- 某基因表达是否随**与肿瘤距离**形成梯度？

---

## 七、NCI 推荐的学习资源

| 资源 | 内容 |
|------|------|
| [NCI ST 培训主页](https://www.cancer.gov/about-nci/organization/cbiit/training/library/st) | 系统入门指南（本文来源） |
| spatialGE Webinar | 结合 SpaGCN、STdeconvolve、InSituType 的实战讲解 |
| spatialGE 介绍视频 | 快速了解平台 |
| spatialGE GitHub | 命令行 / R 包版本 |
| NCI CCR Spatial Biology 页面 | 最新 ST 工具汇总 |
| Human Tumor Atlas Network 数据标准 | 公开 ST 数据资源 |

### 推荐阅读

- spatialGE is a User-Friendly Web Application that Facilitates Spatial Transcriptomics Data Analysis. *Cancer Research*, 2025.
- The Covariance Environment Defines Cellular Niches for Spatial Inference. *Nature Biotechnology*, 2024.

---

## 八、写在最后

NCI 这份指南的核心信息可以浓缩为三句：

1. **ST 的学习曲线陡峭，但值得投入** — 它串联机器学习、图像处理、空间统计，是理解癌症与精准医学的重要工具
2. **先问科学问题，再选成像 vs 测序** — 没有万能平台，只有最匹配的问题
3. **spatialGE 降低了入门门槛** — 零代码也能完成从 QC 到空间域检测、去卷积的完整流程

如果你刚接触空间转录组，不妨从 NCI 培训页 + spatialGE 练手数据开始；已有 Visium / CosMx 数据的，可以直接导入试跑一遍 pipeline。

---

## 参考文献

1. National Cancer Institute, Center for Biomedical Informatics and Information Technology (CBIIT). Spatial Transcriptomics. Training Guide Library. https://www.cancer.gov/about-nci/organization/cbiit/training/library/st

2. spatialGE is a User-Friendly Web Application that Facilitates Spatial Transcriptomics Data Analysis. *Cancer Research*, 2025.

3. The Covariance Environment Defines Cellular Niches for Spatial Inference. *Nature Biotechnology*, 2024.
