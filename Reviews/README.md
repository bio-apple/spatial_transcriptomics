

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

#
### Spot vs Bin 概念速查

| 比较维度 | Spot（物理捕获位点） | Bin（数字分箱/网格） |
|----------|----------------------|----------------------|
| 定义 | 芯片上固定的物理捕获区域 | 测序后按坐标划分的虚拟网格 |
| 典型平台 | Visium v1/v2（55 μm spot） | Visium HD、StrataMap（2 μm / 1 μm 特征 → 8–16 μm bin 分析） |
| 数据特点 | 多细胞混合，需去卷积 | 更细粒度，可细胞分割 |
| 分析策略 | SPOTlight / Cell2location | 直接注释或 Bin2cell |

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