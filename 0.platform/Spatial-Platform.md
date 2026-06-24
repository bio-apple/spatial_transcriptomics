# Comparative summary of key features and capabilities across different spatial platforms

> **文献基础**：Lim H J, Wang Y, Buzdin A, et al. A practical guide for choosing an optimal spatial transcriptomics technology from seven major commercially available options. *BMC Genomics*, 2025, 26(1): 47.  
> DOI：[10.1186/s12864-025-11235-y](https://doi.org/10.1186/s12864-025-11235-y)（开放获取）
---

## 平台分组

| 分组 | 技术类型 | 平台 |
|------|----------|------|
| **G1** | 测序法 · 低分辨率 | 10x Visium、GeoMx DSP |
| **G2** | 测序法 · 高分辨率 | 10x Visium HD、Stereo-seq、**Illumina StrataMap Spatial** ★新增 |
| **G3** | 成像法 · 单细胞/亚细胞 | Vizgen MERSCOPE、10x Xenium、Bruker CosMx SMI |

---

## Table 1（主表）

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

## 2025–2026 关键产品更新摘要

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

- **Human 6K Discovery Panel**：6000 基因 + ≤200 自定义（较原文 6000+200 一致，产品已成熟上市）  
- **Human Whole Transcriptome (WTX) Panel**：~18,000–19,000 蛋白编码基因（成像法准全转录组，2025–2026 新能力）  
- 亚细胞分辨率；FFPE/FF 均支持；AtoMx 数据分析平台

### 其他平台（相对原文变化较小）

| 平台 | 备注 |
|------|------|
| **10x Visium / Visium HD** | 仍为 Visium 生态主力；Visium HD 支持 FF、FFPE、FxF；2 µm 连续网格 |
| **GeoMx DSP** | 现归属 Bruker Spatial Biology；ROI 圈选 + 全转录组/高多重蛋白仍是其核心场景 |
| **Stereo-seq** | DNB 0.22 µm 特征；任意物种；需 DNBSEQ-T7 测序；超大捕获面积（最大可达 cm 级） |
| **MERSCOPE** | 开放定制平台（≤1000 基因无设计费）仍是差异化优势；培养细胞/类器官支持 |

---

## 快速选型提示（结合原文 Table 2 逻辑）

| 你的需求 | 优先考虑 |
|----------|----------|
| 无偏全转录组 + 最快上手 | **10x Visium**（G1） |
| 无偏全转录组 + 已知 ROI / 多活检并行 | **GeoMx**（G1） |
| 无偏全转录组 + 单细胞尺度 + FFPE 成熟生态 | **Visium HD**（G2） |
| 无偏全转录组 + 任意物种 + 超大组织 | **Stereo-seq**（G2） |
| 无偏全转录组 + **1 µm 分辨率 + 大捕获面积 + Illumina 测序栈** | **StrataMap Spatial**（G2）★ |
| 靶向 panel + 亚细胞精度 + FFPE 稳健 | **Xenium**（G3） |
| 靶向 panel + 开放定制任意物种 | **MERSCOPE**（G3） |
| 靶向高多重（6K）或准全转录组成像（WTX ~18K） | **CosMx**（G3） |

---

## 参考文献

1. Lim H J, Wang Y, Buzdin A, Li X. A practical guide for choosing an optimal spatial transcriptomics technology from seven major commercially available options. *BMC Genomics*, 2025, 26(1): 47. https://doi.org/10.1186/s12864-025-11235-3

2. Illumina. StrataMap Spatial Transcriptome product specifications. https://www.illumina.com/products/by-type/sequencing-kits/library-prep-kits/stratamap-spatial.html （访问 2026-06）

3. Illumina. Press release: Illumina launches StrataMap Spatial Solution. 2026-06-08. https://www.illumina.com/company/news-center/press-releases/

4. 10x Genomics. Xenium In Situ Platform. https://www.10xgenomics.com/platforms/xenium （访问 2026-06）

5. Bruker Spatial Biology. CosMx Human 6K Discovery Panel; CosMx Human Whole Transcriptome Panel. https://brukerspatialbiology.com/products/cosmx-spatial-molecular-imager/

6. You Y, Fu Y, Li L, et al. Systematic comparison of sequencing-based spatial transcriptomic methods. *Nat Methods*, 2024, 21(9): 1743–1754.（Stereo-seq 横向扩散等 G2 平台技术细节）
