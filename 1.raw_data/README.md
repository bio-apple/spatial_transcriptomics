# 测序法 Bin 级空间转录组：原始数据到预处理文件清单

> 解读笔记 | 适用 Visium HD、Stereo-seq、Illumina StrataMap 等「测序 + 连续捕获面 + Bin 分析」平台

---

## 摘要

做 Bin 级空间转录组，拿到数据后常问：**FASTQ 算原始数据吗？Space Ranger 的 `outs/` 里每个文件夹是什么？预处理后的 `.h5ad` 里坐标在哪？H&E 和芯片图是一张还是两张？**

本文按 **湿实验原始 → 官方流程输出 → 下游预处理** 三层，把文件清单和常见误区一次梳理。

---

## 一、Bin 和 Spot 不是一回事

| | **Spot（经典 Visium）** | **Bin（Visium HD / Stereo-seq / StrataMap）** |
|---|---|---|
| 硬件 | 离散圆形捕获岛，点间有盲区 | 连续无缝捕获面 |
| 分辨率 | 固定 ~55 μm | 算法可调（如 2/8/16 μm） |
| 列 ID | 固定 spot barcode | 虚拟 grid bin ID |
| 数据量 | ~5,000 列/片 | 10 万～600 万列/片 |
| 典型任务 | 去卷积 | 细胞分割 / 选 bin 大小 |

---

## 二、三层数据：别搞混「原始」指什么

| 层级 | 是什么 | 谁产生 |
|------|--------|--------|
| **L0 测序原始** | FASTQ、BCL、实验端 H&E | 测序仪 / 实验室 |
| **L1 流程输出** | Space Ranger / SAW / DRAGEN 的 `outs/` | 官方 secondary analysis |
| **L2 下游预处理** | 归一化、PCA 后的 `.h5ad` / Seurat 对象 | Scanpy / Seurat / Stereopy |

生信里说的 **raw matrix**，通常指 L1 的 **`raw_feature_bc_matrix`**（未过滤 count），**不是** FASTQ。

---

## 三、L0：湿实验端原始数据

### 3.1 共通文件

| 类型 | 内容 | 用途 |
|------|------|------|
| **FASTQ** | Read1（spatial barcode）+ Read2（cDNA） | 送入官方流程 |
| **参考基因组** | 物种 GTF + FASTA | 比对注释 |
| **样本信息** | 物种、组织、切片厚度 | 流程配置 |

### 3.2 图像：组织图 vs 芯片图（两种不同用途的图）

| | **组织图（H&E）** | **芯片图（HD slide / CytAssist）** |
|---|---|---|
| 看什么 | 细胞形态、病理结构 | 组织在 **捕获芯片** 上的位置 |
| 用途 | 分割、注释、QC | **barcode → 坐标**（生信必需） |
| 坐标系 | 病理扫描仪 | **芯片坐标系** |

**Visium HD 两条路径：**

```
FF 直接上片：
  → 芯片上明场/H&E 图（1 张，定坐标）
  → 可选额外高分辨 H&E（形态分析）

FFPE + CytAssist：
  → H&E 组织图 ①（形态）
  → CytAssist/芯片对齐图 ②（定坐标）
  → 两张图，软件配准后送 Space Ranger
```

> **组织图 = 看形态；芯片图 = 定位置。**

---

## 四、L1：Visium HD（Space Ranger）

`sample/outs/` 核心结构：

```
outs/
├── web_summary.html
├── metrics_summary.csv
├── barcode_mappings.parquet      # barcode ↔ bin ↔ cell 映射
│
├── binned_outputs/               # ★ Bin 分析主入口
│   ├── square_002um/             # 2 μm（最细，极稀疏）
│   ├── square_008um/             # 8 μm（★ 官方推荐）
│   └── square_016um/             # 16 μm（更平滑）
│       每个 bin 目录下：
│       ├── raw_feature_bc_matrix/
│       │   ├── barcodes.tsv.gz   # ★ 此处 barcode = bin ID
│       │   ├── features.tsv.gz
│       │   └── matrix.mtx.gz
│       ├── raw_feature_bc_matrix.h5
│       ├── filtered_feature_bc_matrix/
│       ├── filtered_feature_bc_matrix.h5
│       └── spatial/
│           ├── tissue_positions.parquet
│           ├── scalefactors_json.json
│           ├── tissue_hires_image.png
│           └── tissue_lowres_image.png
│
└── segmented_outputs/            # 细胞分割（可选）
    ├── cell_segmentations.geojson
    ├── filtered_feature_cell_matrix.h5
    └── spatial/
```

### raw vs filtered

| | **raw_feature_bc_matrix** | **filtered_feature_bc_matrix** |
|---|---|---|
| 含义 | 全部 bin × gene count | 去掉低质量 bin/基因后 |
| 下游 | **必须保留**作 count 备份 | 常用作分析起点 |

---

## 五、L1：Stereo-seq 与 StrataMap

### Stereo-seq（SAW）

| 文件 | 说明 |
|------|------|
| `.raw.gef` | 原始分子坐标 + count |
| `.tissue.gef` | 组织范围内 bin 矩阵 |
| `.cellbin.gef` | 细胞分割后 cell-bin 矩阵 |
| `.gem` | 文本：`geneID, x, y, MIDCount, ExonCount` |
| `.ipr` / 图像包 | StereoMap 配套图像 |

Bin 常用档位：bin20（~10 μm）、bin50（~25 μm）、bin100（~50 μm）。

### Illumina StrataMap（DRAGEN + ICM）

| 文件 | 必需？ | 说明 |
|------|--------|------|
| `pipeline-manifest.json` / `tar.gz` / `.h5ad` | 三选一 | 表达 + 空间信息 |
| `.ome.tiff` | ✅ | H&E / 明场组织图 |
| **`contour.csv`** | ⭕ 可选 | 组织边界轮廓，过滤 OCT/背景 |

> Illumina 公开文档尚未完整公布 `contour.csv` 列名 schema；最权威参考是 ICM Demo Data 示例。

---

## 六、L1 空间文件：各平台共通逻辑

| 组件 | 作用 | Visium HD 示例 |
|------|------|----------------|
| **坐标表** | 每个 bin 的 (x,y) | `tissue_positions.parquet` |
| **缩放因子** | 图像像素 ↔ bin 坐标 | `scalefactors_json.json` |
| **组织底图** | 可视化、分割 | `tissue_hires_image.png` |
| **映射表** | bin ↔ cell ↔ nucleus | `barcode_mappings.parquet` |
| **分割边界** | 细胞轮廓 | `.geojson` |

---

## 七、L2：预处理后的文件（`.h5ad` / Seurat）

### 表达矩阵层

| 槽位 | 内容 |
|------|------|
| `X` / counts | 当前使用的矩阵 |
| `layers['counts']` | **原始 count 备份** |
| `layers['normalized']` | 归一化后 |

### 空间坐标（h5ad）

```python
adata.obsm['spatial']    # 每个 bin 的 (x, y)
adata.uns['spatial']     # 图像 + scale factors
```

普通 scRNA-seq 的 h5ad **没有**这些键；Bin ST 专用 h5ad **必须有**才能画空间图。

### Seurat（Visium HD 多 assay）

```r
Load10X_Spatial(data.dir = "outs/", bin.size = c(8, 16))
# → Spatial.008um, Spatial.016um
```

### 通用交换格式

| 格式 | 典型内容 |
|------|----------|
| **`.h5ad`** | AnnData：X + obs + var + obsm + uns |
| **`.rds`** | Seurat 多 assay 对象 |
| **`.cloupe`** | 10x Loupe 浏览器（8 μm） |

---

## 八、完整数据流（Visium HD）

```
FASTQ + 芯片图（+ 可选 H&E）
        ↓  Space Ranger
outs/binned_outputs/square_008um/
  ├── raw_feature_bc_matrix/      ← L1「原始 count」
  ├── filtered_feature_bc_matrix/   ← L1「官方过滤后」
  └── spatial/                      ← 坐标 + 图像
        ↓  Scanpy / Seurat
QC → 归一化 → HVG → PCA → 聚类
        ↓
.h5ad / Seurat object
  ├── obsm['spatial']
  ├── uns['spatial']
  └── layers['counts']
        ↓  可选
Bin2cell / segment → 细胞级矩阵
```

---

## 九、易混概念速查

| 概念 | 和 Bin 原始数据的关系 |
|------|----------------------|
| **多切片对齐**（PASTE/STAligner） | 多张切片之间；**单张不需要** |
| **卷积/GCN**（SpaGCN） | 单张切片内聚合邻域 |
| **是否只有 Spot 才要对齐？** | ❌ 多片 cell-level 同样需要对齐 |

---

## 十、实操 Checklist

**拿到数据先核对：**

- [ ] FASTQ 是否齐全（L0）
- [ ] `binned_outputs/square_008um/` 是否存在（Visium HD）
- [ ] `raw_*` 和 `filtered_*` 是否都有
- [ ] `spatial/tissue_positions.parquet` 是否存在
- [ ] 组织图 vs 芯片图是否配准（CytAssist 流程）
- [ ] StrataMap 是否有 `.ome.tiff`；边界不清时是否需 `contour.csv`
- [ ] 转 h5ad 后是否写入 `obsm['spatial']`

**分析入口建议：**

| 平台 | 推荐起点 |
|------|----------|
| Visium HD | `square_008um/filtered_feature_bc_matrix` |
| Stereo-seq | `.tissue.gef` 或 Stereopy 转 `.h5ad` |
| StrataMap | DRAGEN `tar.gz` 或 ICM 导入 |

---

## 十一、写在最后

1. **L0 FASTQ ≠ 生信 raw matrix**——分析起点是官方流程 `outs/`
2. **组织图和芯片图是两种图**——CytAssist FFPE 通常两张都要
3. **预处理 h5ad 要自带 `obsm['spatial']`**——否则无法完整做 ST 分析

---

## 参考文献

1. 10x Genomics. Space Ranger Output Overview. https://www.10xgenomics.com/support/software/space-ranger

2. Illumina. Supported Data Types — Illumina Spatial Solution. https://help.multiomics.illumina.com/icm/reference/supported-data-types

3. Bioconductor OSTA. Importing Visium HD data. https://bioconductor.org/books/release/OSTA/pages/bkg-importing-data.html

4. Satijalab. Visium HD analysis with Seurat. https://satijalab.org/seurat/articles/visiumhd_analysis_vignette
