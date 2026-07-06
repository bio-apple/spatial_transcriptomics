# CROST：空间转录组综合资源库解读

> 论文解读 | Wang G, Wu S, Xiong Z, et al. *Nucleic Acids Research*, 2024, 52(D1): D882–D890  
> 原文链接：[https://academic.oup.com/nar/article/52/D1/D882/7288834](https://academic.oup.com/nar/article/52/D1/D882/7288834)  
> PMC 全文：[https://pmc.ncbi.nlm.nih.gov/articles/PMC10773281/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10773281/)  
> 数据库地址：[https://ngdc.cncb.ac.cn/crost](https://ngdc.cncb.ac.cn/crost)

---

## 一、研究背景

空间转录组技术能够在组织原位解析基因表达的空间分布，揭示细胞微环境与组织结构的分子基础。然而，空间转录组数据分散在 NCBI、EBI、10x Genomics 等多个平台，现有数据库在数据规模、分析功能和单细胞整合方面存在明显不足。

中国科学院北京基因组研究所（国家基因组科学中心）王国亮、吴松等人开发了 **CROST**（Comprehensive Repository Of Spatial Transcriptomics），旨在构建一个集数据存储、标准化分析、交互可视化与在线工具于一体的空间转录组综合资源库。

---

## 二、现有数据库的局限

| 数据库 | 主要局限 |
|--------|----------|
| SpatialDB | 仅 24 个数据集，无空间分析功能 |
| SODB / STomicsDB | 有交互可视化，但缺少空间分析 |
| Aquila | 无组织图像信息，未整合单细胞数据 |
| SPASCER | 缺少交互式空间可视化与在线分析平台 |

CROST 的定位是：**不仅存储数据，还提供完整的标准化分析链路与在线工具**。

---

## 三、数据规模与来源

### 3.1 收录概况

| 指标 | 数量 |
|------|------|
| 空间转录组项目 | 182 |
| 子数据集/样本 | 1,033 |
| 物种 | 8（人、小鼠、斑马鱼、鸡、狗、兰花、大鼠等） |
| 组织类型 | 35 |
| 疾病类型 | 56 |
| 肿瘤相关 SVG | 48,043 |
| 注释细胞类型 | 168 |

### 3.2 数据来源

数据从以下公共数据库采集：

- NCBI
- EBI
- CNCB（中国国家生物信息中心）
- Broad Institute Single Cell Portal
- 10x Genomics
- DDBJ

并通过 PubMed、bioRxiv 文献检索补充。筛选标准包括：具备原始测序文件，以及空间坐标或图像文件。

---

## 四、标准化分析流程

CROST 对原始测序数据执行统一的预处理与分析：

```
原始测序数据
    ↓
STAR 比对 → 质控过滤（去除低质量/重复 reads）
    ↓
Harmony 批次校正（多样本数据集）
    ↓
BayesSpace 空间聚类（qTune 确定最优簇数）
    ↓
SPARK 识别空间可变基因（SVG，adj.P < 0.01）
    ↓
GO / KEGG 富集分析（clusterProfiler）
    ↓
SPOTlight 细胞类型去卷积（整合 scRNA-seq）
    ↓
细胞类型相关性与共定位分析（SPOTlight）
    ↓
CellChat 细胞-细胞通讯分析
    ↓
ssGSEA 单样本基因集富集
```

### 主要工具

| 分析步骤 | 工具 |
|----------|------|
| 序列比对 | STAR |
| 批次校正 | Harmony |
| 空间聚类 | BayesSpace |
| SVG 识别 | SPARK |
| 功能富集 | clusterProfiler |
| 细胞类型注释 | SPOTlight |
| 细胞通讯 | CellChat |
| 基因集富集 | GSVA / ssGSEA |

---

## 五、四大功能模块

### 5.1 Browse（浏览模块）

**功能**：高效浏览数据集，查看元信息与预分析结果。

**元数据标准**：

- 项目级：22 项元数据（数据来源、实验设计、组织/细胞系、疾病状态等）
- 样本级：38 项结构化元数据（基本信息、样本特征、生物学状态、实验方案、质量评估）

**每个样本的 7 类预分析结果**：

1. 数据概览（count、gene、线粒体/核糖体统计的空间分布）
2. 降维与聚类（BayesSpace 空间聚类 + ssGSEA 富集）
3. 空间可变基因（SPARK + GO/KEGG 富集）
4. 细胞类型注释（SPOTlight 去卷积）
5. 细胞类型相关性与共定位
6. 簇-簇通讯分析（CellChat）
7. 细胞-细胞通讯分析（CellChat）

**访问**：[https://ngdc.cncb.ac.cn/crost/browse/](https://ngdc.cncb.ac.cn/crost/browse/)

### 5.2 Cancer SVG（肿瘤空间可变基因模块）

**功能**：整合多组学数据，探索肿瘤相关空间可变基因。

**数据规模**：

- 48,043 个 SVG
- 主要富集：肾癌（8,323）、肝癌（6,380）、黑色素瘤（5,964）

**整合组学层次**：

- 空间转录组（原位表达）
- 经典转录组（表达水平）
- 表观组（DNA 甲基化）
- 基因组（拷贝数变异 CNV）
- 临床预后（生存分析）

**案例**：肝癌 SVG 基因 AATF 高表达与患者预后不良显著相关，与已有研究一致。

**访问**：[https://ngdc.cncb.ac.cn/crost/cancer-svg](https://ngdc.cncb.ac.cn/crost/cancer-svg)

### 5.3 Explore（探索模块）

**功能**：交互式空间数据可视化与探索。

**主要特性**：

- 基于 Cirrocumulus 的交互式 UMAP 与空间切片可视化
- 同时展示空间转录组与配对单细胞数据
- 在线探索细胞通讯网络（信号通路、发送者/接收者角色）
- 空间分布可视化特定信号通路的细胞群体
- 检索细胞类型共定位与相关性结果

**验证案例**：在结直肠癌样本中，FAP+ 成纤维细胞与巨噬细胞呈现强相关性和显著空间重叠，与 Su et al. (Nat Commun, 2022) 结果一致。

**访问**：[https://ngdc.cncb.ac.cn/crost/analyze/spatial-explorer](https://ngdc.cncb.ac.cn/crost/analyze/spatial-explorer)

### 5.4 Online Analysis（在线分析模块）

面向无编程基础用户，提供两个在线工具：

#### ssGSEA

- 上传原始 count 矩阵
- 对 7 类基因集进行单样本富集分析：
  - 癌症生物学过程
  - 细胞状态
  - 染色体细胞遗传学条带
  - Gene Ontology
  - KEGG 通路
  - MicroRNA 靶基因
  - 转录因子靶基因
- 支持打包下载全部结果

**访问**：[https://ngdc.cncb.ac.cn/crost/tools/ssgsea](https://ngdc.cncb.ac.cn/crost/tools/ssgsea)

#### SpatialAP（一站式空间分析平台）

涵盖完整分析链路：

- 质控
- 降维聚类
- SVG 分析
- 细胞类型注释（整合 scRNA-seq）
- 共定位与相关性分析
- 细胞通讯分析
- 生物学功能富集

**三种分析模式**：

| 模式 | 说明 |
|------|------|
| 模式 1 | 上传 scRNA-seq + 从 CROST 选择空间数据 |
| 模式 2 | 上传空间数据 + 从 CROST 选择 scRNA-seq |
| 模式 3 | 同时上传空间与单细胞数据 |

分析完成后通过邮件通知，结果在 CROST 平台查看。私有数据本地处理，保障数据安全。

**访问**：[https://ngdc.cncb.ac.cn/crost/tools/spatial-pipeline](https://ngdc.cncb.ac.cn/crost/tools/spatial-pipeline)

---

## 六、技术架构

| 组件 | 技术 |
|------|------|
| 前端 | Vue3.js + Vite |
| 后端 API | Node.js（CentOS 部署） |
| 数据库 | MongoDB |
| 可视化 | Echarts、Cirrocumulus |
| 数据分析 | R 4.1 |

---

## 七、与相关工具的对比

### 7.1 与现有空间转录组数据库

| 维度 | SpatialDB | Aquila | SPASCER | CROST |
|------|-----------|--------|---------|-------|
| 数据集数量 | 24 | 较多 | 较多 | 182 项目 / 1033 样本 |
| 空间分析 | 无 | 有 | 有 | 完整 pipeline |
| 交互可视化 | 有限 | 有限 | 有限 | Cirrocumulus 交互式 |
| 单细胞整合 | 无 | 无 | 部分 | SPOTlight 去卷积 |
| 肿瘤 SVG 模块 | 无 | 无 | 无 | 48,043 SVG + 多组学 |
| 在线分析工具 | 无 | 无 | 无 | ssGSEA + SpatialAP |

### 7.2 与 SpatialQC 的互补关系

| 工具 | 定位 | 发表期刊 |
|------|------|----------|
| SpatialQC | 空间转录组数据质控与清洗 | Bioinformatics (btae458) |
| CROST | 数据整合、浏览、预分析、在线分析 | NAR (gkad782) |

推荐工作流：**SpatialQC 质控 → CROST / SpatialAP 下游分析**

---

## 八、适用人群

- 需要快速检索、下载公开空间转录组数据的研究者
- 希望参考标准化分析结果（聚类、SVG、去卷积、通讯）的用户
- 从事肿瘤研究、关注空间可变基因与多组学整合的临床与基础研究人员
- 无编程基础、需在线分析工具的实验生物学家
- 需要同时对比空间数据与配对单细胞数据的研究者

---

## 九、未来发展方向

- 整合高分辨率空间多组学数据（代谢组、甲基化组等）
- 增强与其他组学数据库的互联互通
- 持续扩充数据集规模
- 作为 NGDC 开放资源持续更新维护

---

## 十、总结

CROST 是国家基因组科学中心（NGDC）空间组学资源的重要组成部分，核心优势包括：

1. **数据全面**：182 项目、1033 样本，覆盖 8 物种、35 组织、56 疾病
2. **分析深入**：预置完整空间分析 pipeline（聚类、SVG、去卷积、通讯）
3. **肿瘤专精**：48,043 肿瘤 SVG + 转录组/表观组/基因组多组学整合
4. **易于使用**：交互式可视化 + 免编程在线工具（ssGSEA、SpatialAP）
5. **免费开放**：作为 NGDC 公共资源持续维护

---

## 参考文献

```
Wang G, Wu S, Xiong Z, Qu H, Fang X, Bao Y.
CROST: a comprehensive repository of spatial transcriptomics.
Nucleic Acids Research. 2024 Jan 5;52(D1):D882-D890.
doi: 10.1093/nar/gkad782
```

**相关链接**

- 论文（Oxford Academic）：https://academic.oup.com/nar/article/52/D1/D882/7288834
- PMC 全文：https://pmc.ncbi.nlm.nih.gov/articles/PMC10773281/
- 数据库主页：https://ngdc.cncb.ac.cn/crost
- Browse 模块：https://ngdc.cncb.ac.cn/crost/browse/
- Cancer SVG：https://ngdc.cncb.ac.cn/crost/cancer-svg
- Explore 模块：https://ngdc.cncb.ac.cn/crost/analyze/spatial-explorer
- ssGSEA 工具：https://ngdc.cncb.ac.cn/crost/tools/ssgsea
- SpatialAP 工具：https://ngdc.cncb.ac.cn/crost/tools/spatial-pipeline
