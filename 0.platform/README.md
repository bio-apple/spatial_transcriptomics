# 空间转录组平台选型与分析指南

> **文献基础**：Lim H J, Wang Y, Buzdin A, et al. A practical guide for choosing an optimal spatial transcriptomics technology from seven major commercially available options. *BMC Genomics*, 2025, 26(1): 47.  
> DOI：[10.1186/s12864-025-11235-y](https://doi.org/10.1186/s12864-025-11235-y)（开放获取）  
> **分析框架**：Fred Hutch *Choosing Genomics Tools* Ch.14 — [Spatial transcriptomics](https://hutchdatascience.org/Choosing_Genomics_Tools/spatial-transcriptomics-1.html)

---

## 摘要

单细胞 RNA-seq 能告诉我们组织里有哪些细胞，却回答不了：**这些细胞在哪里、如何组织成微环境、又在和谁对话？**

空间转录组（Spatial Transcriptomics, ST）正是为补上这块拼图。本文整合两部分内容：

1. **实验选型** — BMC Genomics 2025 七大商业平台横向对比（G1/G2/G3 分组 + Table 1 规格表）
2. **分析落地** — Fred Hutch 课程的分析逻辑、质控要点与工具地图

形成「先选平台、再跑流程」的实用框架。

---

## 一、核心权衡：空间分辨率 vs 分子分辨率

Fred Hutch 课程强调，几乎所有 ST 技术都面临同一组矛盾：

| 方向 | 代表平台 | 特点 |
|------|----------|------|
| **高分子分辨率**（全转录组） | Visium、Visium HD、Stereo-seq、StrataMap | 无偏发现，但以 spot/ROI/bin 为单位 |
| **高空间分辨率**（单细胞/亚细胞） | Xenium、CosMx、MERSCOPE | 定位精准，但多为靶向基因 panel |

**没有「又全转录组、又单细胞、又便宜」的完美方案——科学问题决定平台，平台决定分析策略。**

---

## 二、平台分组（BMC Genomics 2025）

| 分组 | 技术类型 | 平台 |
|------|----------|------|
| **G1** | 测序法 · 低分辨率 | 10x Visium、GeoMx DSP |
| **G2** | 测序法 · 高分辨率 | 10x Visium HD、Stereo-seq、**Illumina StrataMap Spatial** ★ |
| **G3** | 成像法 · 单细胞/亚细胞 | Vizgen MERSCOPE、10x Xenium、Bruker CosMx SMI |

### G1：测序法 · 低分辨率（spot/ROI 级）

| 平台 | 分辨率 | 基因覆盖 | 适合场景 |
|------|--------|----------|----------|
| **10x Visium** | 55 μm spot（多细胞） | 全转录组 | 最快上手、无偏发现、FF/FFPE |
| **GeoMx DSP** | >55 μm ROI（建议 ≥50 细胞） | ROI 内全转录组 + 570 蛋白 | 病理圈选区域、多活检并行 |

### G2：测序法 · 高分辨率（单细胞尺度）

| 平台 | 分辨率 | 基因覆盖 | 适合场景 |
|------|--------|----------|----------|
| **Visium HD** | 2 μm 连续网格（分析常用 8/16 μm bin） | 全转录组 | FFPE 成熟生态 + 近单细胞精度 |
| **Stereo-seq** | 0.22 μm DNB 特征 | 全转录组 | 任意物种、超大捕获面积（cm 级） |
| **Illumina StrataMap** ★ | 1 μm 连续特征 + 细胞分割 | 全转录组（polyA⁺） | 大捕获面积（最大 50×15 mm）、Illumina 测序栈 |

### G3：成像法 · 单细胞/亚细胞

| 平台 | 分辨率 | 基因覆盖 | 适合场景 |
|------|--------|----------|----------|
| **10x Xenium** | ~200 nm | Prime 5K ~5000 基因；定制 ~480 基因 | FFPE 稳健、同片 RNA+蛋白（2025–2026） |
| **CosMx SMI** | 亚细胞 | 6K panel 或 WTX ~18K 蛋白编码基因 | 高多重靶向 / 准全转录组成像 |
| **MERSCOPE** | 单细胞/亚细胞 | 预制 ~5000 或全定制 ≤1000 基因 | 开放定制任意物种、支持类器官 |

---

## 三、Table 1：平台规格主表

| Key Features | **10x Visium** | **GeoMx DSP** | **10x Visium HD** | **Stereo-seq** | **Illumina StrataMap Spatial** ★ | **MERSCOPE** | **10x Xenium** | **CosMx SMI** |
|:---|:---|:---|:---|:---|:---|:---|:---|:---|
| **Group** | G1 | G1 | G2 | G2 | G2 | G3 | G3 | G3 |
| **Technology** | 测序 · 空间条形码阵列 | 测序 · ROI 探针释放 + 条形码 | 测序 · 探针杂交捕获 | 测序 · DNB 纳米球阵列 | 测序 · poly(A) 捕获 + 1 µm 特征 | 成像 · MERFISH 二进制条形码 | 成像 · ISS/ISH 混合（padlock + RCA） | 成像 · smFISH 循环解码 |
| **Suitable species** | V1：任意物种；V2（CytAssist）：人、小鼠 | 人、小鼠 | 人、小鼠 | 任意物种 | 真核生物（物种通用） | 预制 panel：人、小鼠；**全定制 ≤1000 基因：任意物种，无设计费** | 预制 panel：人、小鼠；全定制 panel 可行但设计费高 | 预制 panel：人、小鼠；全定制 panel 可行但设计费高 |
| **Applicable tissue types** | FF、FFPE | FF、FFPE | FF、FFPE、FxF（固定冷冻） | FF、FFPE、FxF | **FF（OCT 包埋）**；FFPE 版本开发中（预计 2027 年早期客户启用） | FF、FFPE、**培养细胞/类器官** | FF、FFPE | FF、FFPE |
| **Capture (imaging) area / slide** | V1：4 × 6.5 × 6.5 mm；V2：2 × 6.5 × 6.5 mm | 36 × 14 mm（大视野 ROI 选择） | 2 × 6.5 × 6.5 mm | 10 × 10 mm 或 20 × 30 mm（更大芯片可选） | **大规格：50 × 15 mm（750 mm²）；标准：17 × 15 mm（255 mm²）**；每片 1–12 张切片 | 18 × 22 mm | 10.45 × 22.45 mm（连续扫描） | 15 × 20 mm（0.5 × 0.5 mm FOV 组合） |
| **Number of genes profiled** | 全转录组（无偏） | 全转录组（ROI 内） | 全转录组（无偏） | 全转录组（无偏） | **全转录组（poly(A) 编码 + 长链非编码 RNA）** | 预制 panel 最高 ~5000 + ~100 自定义；**全定制 ≤1000 基因** | **Xenium Prime 5K：~5000 基因 + ≤100 自定义**；预制 panel 377–5000 基因；**独立定制 panel 最高 ~480 基因** | **6K Discovery：6000 基因 + ≤200 自定义**；**Human WTX：~18,000–19,000 蛋白编码基因**；1K 等预制 panel 可选 |
| **Number of proteins profiled** | 35（RNA/蛋白分片顺序检测） | 570（RNA/蛋白分片顺序检测） | 暂不支持 | 暂不支持 | 暂不支持 | **6（RNA/蛋白同片共成像）** | **Xenium Protein：同片 RNA + 蛋白（免疫肿瘤等预制 subpanel，2025–2026 上市）** | 68（RNA/蛋白分片顺序检测） |
| **Spatial resolution** | 55 µm（多细胞 spot） | > 55 µm（ROI，厂商建议 ≥50 细胞/ROI） | 单细胞尺度（2 µm 连续特征，无间隙） | 单细胞（0.22 µm DNB 特征；中心距 ~0.5 µm） | **细胞分辨率（1 µm 连续表面特征 + 整合细胞分割）** | 单细胞 / 亚细胞 | 单细胞 / 亚细胞（~200 nm 光学分辨率） | 单细胞 / 亚细胞 |
| **Hands-on time** | 1–2 天 | ~3 天 | ~2 天 | 2–3 天 | **~8.5 h（实验操作）；总流程 ~2.5 天（至测序）** | 4–5 天 | ~3 天 | ~2 天 |

**注释**  
- *FF*：Fresh Frozen（新鲜冷冻）；*FFPE*：福尔马林固定石蜡包埋；*FxF*：Fixed Frozen（固定冷冻）

---

## 四、2025–2026 关键产品更新

### Illumina StrataMap Spatial（新增，G2）

| 项目 | 最新信息 |
|------|----------|
| 发布状态 | 2026 年 6 月正式商业化（Research Use Only） |
| 核心定位 | 测序法、无偏全转录组、**1 µm 特征密度**、大捕获面积 |
| 样本 | 当前仅 **FF**；FFPE 版本开发中（目标 2027 年客户启用） |
| 捕获面积 | 50 × 15 mm（750 mm²）或 17 × 15 mm；支持多切片并行 |
| 灵敏度（官方开发数据） | 10 × 10 µm bin 最高 ~4000 unique transcripts；检测基因数约为同类 panel 平台 **>2×** |

### 10x Xenium（G3 更新）

- **Xenium Prime 5K**：~5000 基因预制 panel，可添加 ≤100 自定义基因  
- **Xenium Protein**（2025–2026 上市）：同切片 RNA + 蛋白共检，免疫肿瘤等预制 subpanel  
- 独立定制 panel 最高 ~480 基因；预制 panel 覆盖 377–5000 基因  
- 扫描面积 ~10.45 × 22.45 mm；FFPE 降解样本表现突出（多篇 benchmark 证实）

### Bruker CosMx SMI（G3 更新）

- **Human 6K Discovery Panel**：6000 基因 + ≤200 自定义  
- **Human Whole Transcriptome (WTX) Panel**：~18,000–19,000 蛋白编码基因（成像法准全转录组，2025–2026 新能力）  
- 亚细胞分辨率；FFPE/FF 均支持；AtoMx 数据分析平台

### 其他平台

| 平台 | 备注 |
|------|------|
| **10x Visium / Visium HD** | 仍为 Visium 生态主力；Visium HD 支持 FF、FFPE、FxF；2 µm 连续网格 |
| **GeoMx DSP** | 现归属 Bruker Spatial Biology；ROI 圈选 + 全转录组/高多重蛋白仍是其核心场景 |
| **Stereo-seq** | DNB 0.22 µm 特征；任意物种；需 DNBSEQ-T7 测序；超大捕获面积（最大可达 cm 级） |
| **MERSCOPE** | 开放定制平台（≤1000 基因无设计费）仍是差异化优势；培养细胞/类器官支持 |

---

## 五、快速选型

| 你的需求 | 优先考虑 |
|----------|----------|
| 无偏全转录组 + 最快上手 | **10x Visium**（G1） |
| 无偏全转录组 + 已知 ROI / 多活检并行 | **GeoMx**（G1） |
| 无偏全转录组 + 单细胞尺度 + FFPE 生态成熟 | **Visium HD**（G2） |
| 无偏全转录组 + 任意物种 + 超大组织 | **Stereo-seq**（G2） |
| 无偏全转录组 + **1 µm 分辨率 + 大捕获面积 + Illumina 测序栈** | **StrataMap Spatial**（G2）★ |
| 靶向 panel + 亚细胞精度 + FFPE 稳健 | **Xenium**（G3） |
| 靶向 panel + 开放定制任意物种 | **MERSCOPE**（G3） |
| 靶向高多重（6K）或准全转录组成像（WTX ~18K） | **CosMx**（G3） |

### 样本类型决策补充

| 样本类型 | 推荐平台 |
|----------|----------|
| **FFPE 归档样本** | Visium v2/CytAssist、Visium HD、Xenium、CosMx |
| **新鲜冷冻 FF** | Visium、Stereo-seq、StrataMap（当前仅 FF） |
| **非模式物种** | Visium（polyA⁺）、Stereo-seq（定制探针）、MERSCOPE（≤1000 基因无设计费）、StrataMap（polyA⁺ 真核通用） |

### Spot vs Bin 概念速查

| 比较维度 | Spot（物理捕获位点） | Bin（数字分箱/网格） |
|----------|----------------------|----------------------|
| 定义 | 芯片上固定的物理捕获区域 | 测序后按坐标划分的虚拟网格 |
| 典型平台 | Visium v1/v2（55 μm spot） | Visium HD、StrataMap（2 μm / 1 μm 特征 → 8–16 μm bin 分析） |
| 数据特点 | 多细胞混合，需去卷积 | 更细粒度，可细胞分割 |
| 分析策略 | SPOTlight / Cell2location | 直接注释或 Bin2cell |

---

## 六、空间转录组能回答什么科学问题？

（来源：Fred Hutch 课程 + Nature/Genome Medicine 综述）

1. **描绘细胞邻域与组织域** — 哪些细胞类型在何处聚集？肿瘤-间质边界如何组织？
2. **发现空间调控过程** — 哪些基因/通路仅在特定区域活跃？（发育梯度、信号通路空间限制）
3. **验证细胞通讯假说** — 在真实共定位背景下检验配体-受体相互作用
4. **整合组织形态学** — 表达模式与 H&E、IF 图像直接对照
5. **生物标志物与药物靶点** — 区域特异性表达特征，辅助靶点发现

---

## 七、标准分析流程与质控

```
① 样本制备（5–10 μm 切片，FF / FFPE / OCT）
      ↓
② RNA 捕获或杂交（测序法：空间条码；成像法：荧光探针）
      ↓
③ 定量（NGS 测序 + 条码解码 / 循环成像计数）
      ↓
④ 质控与预处理
      ↓
⑤ 可视化（UMAP + 空间热图 + H&E 对照）
      ↓
⑥ 聚类、域识别、去卷积、细胞通讯
```

### 质控要点（Fred Hutch 特别强调）

- ST 数据**零值多、dropout 高**，需过滤低表达基因和低 count spot/bin
- **注意**：细胞稀疏区域 count 天然偏低，勿误删生物学变异
- 线粒体/核糖体比例可反映组织坏死
- Spot 级数据（Visium）常需 **去卷积**；Bin 级数据（Visium HD）可选不同 bin 大小或细胞分割
- **建议**：邀请病理或组织学专家校验聚类/注释结果

---

## 八、优势与局限：理性预期

### 优势

- 保留**真实空间语境**，在组织架构中研究表达与互作
- 可整合**影像数据**（H&E、IF、mIF）
- 细胞通讯分析更可靠（有共定位信息）
- 揭示**空间限制性**生物学过程
- 连接表达模式与病理区域，利于 biomarker 发现

### 局限

| 局限 | 说明 |
|------|------|
| 分辨率权衡 | 全转录组 vs 单细胞精度难以兼得 |
| 技术变异 | 组织处理、捕获、测序/成像各步骤引入批次 |
| 零膨胀 | 探针偏好、低丰度转录本漏检 |
| 分析复杂 | 需专门流程，门槛高于 bulk/scRNA-seq |
| 成本与时间 | 实验 + 测序/成像费用高；G3 平台操作 2–5 天 |

---

## 九、分析工具地图（按环节选型）

| 环节 | 工具 | 适用 |
|------|------|------|
| 原始数据处理 | Space Ranger、GeomxTools | 10x / GeoMx |
| 探索可视化 | Seurat、Squidpy、Giotto、Loupe | R / Python |
| 无代码 | spatialGE-web | 入门友好 |
| 质控 | SpatialQC | 空间专用 QC |
| 组织域识别 | SpaGCN、GraphST、BayesSpace | 整合空间坐标 ± 图像 |
| 空间可变基因 | SPARK、SpatialDE | 区域特异性表达 |
| 细胞去卷积 | SPOTlight、Cell2location、RCTD | Spot 级数据必备 |
| 细胞分割（HD） | Bin2cell、Space Ranger segment | 2 μm bin → 单细胞 |
| 大样本子抽样 | Seurat SketchData、spatially smoothed leverage | Visium HD / atlas 尺度 |
| 切片对齐/3D | PASTE2、Spateo、SABench 评测 | 多切片整合 |
| 细胞通讯 | CellChat（空间版）、NICHES | L-R + 距离 |
| 数据资源 | CROST、STomicsDB | 公开数据浏览与在线分析 |

### 新手推荐路径

| 平台 | 推荐流程 |
|------|----------|
| **Visium** | Space Ranger → Seurat → SPOTlight 去卷积 |
| **Visium HD** | Space Ranger（选 8 μm bin）→ Seurat / Scanpy → 可选 Bin2cell |
| **Xenium / CosMx** | 官方 pipeline → Seurat / Squidpy → 直接细胞级注释 |

---

## 十、整合决策框架：从立项到分析

```
Step 1  明确科学问题
        ├─ 无偏发现？ → G1/G2 测序法
        ├─ 已知区域比较？ → GeoMx
        ├─ 单细胞精度验证？ → G3 成像法
        └─ 3D 重建/多切片？ → 预留对齐分析（PASTE2/Spateo）

Step 2  评估样本条件
        ├─ FFPE 归档 → Visium HD / Xenium / CosMx
        ├─ 新鲜组织 → Visium / Stereo-seq / StrataMap
        └─ 非模式物种 → Visium polyA⁺ / MERSCOPE 定制 / Stereo-seq

Step 3  权衡预算与通量
        ├─ 预算有限、快速探索 → Visium（G1）
        ├─ 大组织、大队列 → StrataMap / Stereo-seq 大面积
        └─ 精准验证 → Xenium / CosMx panel

Step 4  设计分析流程
        ├─ 质控（SpatialQC）
        ├─ 可视化 + 域识别
        ├─ Spot 数据 → 去卷积；Bin/Cell 数据 → 直接注释
        └─ 细胞通讯 + 与 scRNA-seq 整合

Step 5  验证
        └─ 病理专家 + 独立样本 + 功能实验
```

---

## 写在最后

空间转录组正在从「能做实验」走向「能系统解读」。两条线索可以帮你少走弯路：

- **实验选型**：BMC Genomics 2025 的 G1/G2/G3 框架 + 样本/物种/分辨率三维决策
- **分析落地**：Fred Hutch 的目标—流程—工具地图 + 质控与病理验证

> **平台选错 → 数据先天局限，分析难以补救**  
> **分析选错 → 好数据也可能得出错误生物学结论**

两者兼顾，才是空间组学项目的正确打开方式。

---

## 参考文献

1. Lim H J, Wang Y, Buzdin A, Li X. A practical guide for choosing an optimal spatial transcriptomics technology from seven major commercially available options. *BMC Genomics*, 2025, 26(1): 47. https://doi.org/10.1186/s12864-025-11235-y

2. Fred Hutch Data Science Lab. *Choosing Genomics Tools*, Chapter 14: Spatial transcriptomics. https://hutchdatascience.org/Choosing_Genomics_Tools/spatial-transcriptomics-1.html

3. Rao A, Barkley D, França G S, et al. Exploring tissue architecture using spatial transcriptomics. *Nature*, 2021, 596(7871): 211-220.

4. Williams C G, Lee H J, Asatsuma T, et al. An introduction to spatial transcriptomics for biomedical research. *Genome Medicine*, 2022, 14(1): 68.

5. You Y, Fu Y, Li L, et al. Systematic comparison of sequencing-based spatial transcriptomic methods. *Nat Methods*, 2024, 21(9): 1743–1754.

6. Illumina. StrataMap Spatial Transcriptome product specifications. https://www.illumina.com/products/by-type/sequencing-kits/library-prep-kits/stratamap-spatial.html

7. Illumina. Press release: Illumina launches StrataMap Spatial Solution. 2026-06-08. https://www.illumina.com/company/news-center/press-releases/

8. 10x Genomics. Xenium In Situ Platform. https://www.10xgenomics.com/platforms/xenium

9. Bruker Spatial Biology. CosMx Spatial Molecular Imager. https://brukerspatialbiology.com/products/cosmx-spatial-molecular-imager/
