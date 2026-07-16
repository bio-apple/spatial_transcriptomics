

## 二、现有工具的不足

论文指出，空间转录组质控至少有三类需求是单细胞 QC 工具难以覆盖的：

| 需求 | 说明 |
|------|------|
| **空间异常检测** | 低质量 spot/细胞可能集中在特定区域，需将 QC 指标映射到空间坐标上才能发现 |
| **切片级评估** | 3D 或连续切片实验中，需比较、筛选质量差的切片 |
| **平台差异化参数** | Visium、Stereo-seq、MERFISH、Slide-seq 等平台差异大，过滤阈值不能一刀切 |

---

## 三、SpatialQC 简介

**SpatialQC** 是一款基于 Python 的开源工具，目标是**一键完成质量评估、数据清洗和报告生成**。

### 支持的数据格式

- AnnData（Scanpy / Squidpy）
- Seurat（R）
- SingleCellExperiment / SpatialExperiment（Bioconductor）
- gem（Stereo-seq）

### 支持的平台

| 平台类型 | 代表技术 |
|----------|----------|
| 测序类（spot 级） | Visium、ST、Slide-seq、DBiT-seq |
| 测序类（高分辨率） | Stereo-seq、Seq-scope、Pixel-seq、HDST、Visium HD |
| 成像类（单细胞级） | MERFISH、Xenium、CosMx、HybISS |

---

## 四、核心工作流程

SpatialQC 包含三个主要模块：**细胞/spot 评分 → 数据过滤 → 报告生成**。

### 4.1 细胞/spot 评分

对每个捕获单元综合评估以下指标：

- 线粒体基因比例
- 检测到的基因数（`n_genes`）
- UMI 总数（`n_counts`）
- 标记基因检出比例
- 双联体评分（Scrublet）

评分结果可在 QC 报告中以**空间分布图**展示，用于发现局部异常。

### 4.2 三层递进过滤

```
输入数据
    ↓
【切片级过滤】剔除整体质量差的切片（中位得分 < 5，可调）
    ↓
【细胞/spot 级过滤】按 min_genes 过滤低深度单元；去除双联体和高线粒体比例细胞
    ↓
【基因级过滤】过滤在过少细胞中表达的基因，尽量保留标记基因
    ↓
输出：过滤后的 AnnData + HTML 交互式报告
```

| 过滤层级 | 主要参数 | 默认策略示例 |
|----------|----------|--------------|
| 切片级 | `min_score` | 切片中位细胞得分 < 5 则剔除 |
| 细胞/spot 级 | `min_genes`, `mito_percent` | Stereo-seq 建议保留 >70% 细胞；线粒体比例默认 <10% |
| 基因级 | `min_cells` | 尽量保留 >99% 标记基因 |

不同平台提供预设参数组合，用户也可手动调整。

### 4.3 交互式 HTML 报告

报告包含三部分内容：

1. **数据概览**：细胞/spot 数、基因数、切片数等统计信息
2. **质量可视化**：基因数、线粒体比例、标记基因比例等指标的空间分布图与统计图
3. **参数敏感性分析**：不同 `min_genes` / `min_cells` 组合对保留细胞数和标记基因数的影响

支持按切片切换查看，帮助用户在正式分析前确定过滤参数。

---

## 五、关键结果

### 5.1 果蝇胚胎 Stereo-seq 数据

- 数据：16 张切片，15,295 个细胞，13,668 个基因（StomicsDB, STDS0000060）
- 切片 S14 整体得分最低，空间图显示测序深度明显偏低
- 切片 S15 低分区域呈局部聚集，提示可能存在局部技术问题
- 设置 `min_genes = 490`，在保留 >70% 细胞的同时提升数据质量

### 5.2 小鼠胚胎 Slide-seq 数据

- 原始数据基因数、UMI 分布呈双峰
- 大量低深度「细胞」实际来自非组织区域的 bead
- 过滤 `min_genes < 200` 后双峰消失，数据分布更合理

### 5.3 多平台验证

在 Visium、Stereo-seq、MERFISH、Slide-seq 等平台上，过滤后的：

- 细胞质量得分显著提升
- 标签迁移（label transfer）得分显著提升

---

## 六、使用方法

### 6.1 安装

```bash
pip install spatialQC
```

或从 GitHub 克隆：

```bash
git clone https://github.com/mgy520/spatialQC.git
cd spatialQC
pip install -e .
```

### 6.2 基本命令

```bash
SpatialQC --adata your_data.h5ad \
          --0.platform Visium \
          --slice slice \
          --markers markers.csv \
          --mito 'MT-' \
          --ribo 'RPS,RPL' \
          --mito_percent 0.3
```

### 6.3 主要参数说明

| 参数 | 说明 |
|------|------|
| `--adata` / `--input` | 输入文件路径（.h5ad 或 .rds） |
| `--platform` | 平台名称，自动加载推荐阈值 |
| `--slice` | 切片信息所在列名 |
| `--markers` | 标记基因 CSV 文件路径 |
| `--species` / `--tissue_class` / `--tissue_type` | 从 CellMarker 2.0 数据库自动获取标记基因 |
| `--mito` | 线粒体基因命名模式 |
| `--mito_percent` | 线粒体基因比例上限 |
| `--min_genes` | 每个细胞最少基因数（可自动推断） |
| `--min_cells` | 每个基因最少检出细胞数（可自动推断） |
| `--doublet` | 是否进行双联体检测 |

### 6.4 下游分析示例

```python
import scanpy as sc
import GraphST

# 读取过滤后的数据
adata = sc.read_h5ad('filtered.h5ad')
adata = adata[adata.obs.slice == '151673']

# 空间域识别
model = GraphST.GraphST(adata, device='cuda')
adata = model.train()
```

---

## 七、与同类工具的比较

| 维度 | 单细胞 QC 工具（scQCEA、popsicleR） | SpatialQC |
|------|-------------------------------------|-----------|
| 适用数据 | 单细胞 RNA-seq | 空间转录组 |
| 空间异常检测 | 不支持 | 支持（空间分布图） |
| 切片级比较 | 不支持 | 支持 |
| 平台预设参数 | 有限 | 多平台预设 |
| 输出报告 | 有 | 交互式 HTML 报告 |
| 输入格式 | 主要 Cell Ranger 输出 | AnnData / Seurat / SCE 等 |

---


---

## 参考文献

```
Mao G, Yang Y, Luo Z, et al. SpatialQC: automated quality control for spatial transcriptome data.Bioinformatics. 2024 Aug;40(8):btae458.doi: 10.1093/bioinformatics/btae458
```

**相关链接**

- 论文（Oxford Academic）：https://academic.oup.com/bioinformatics/article/40/8/btae458/7720780
- PMC 全文：https://pmc.ncbi.nlm.nih.gov/articles/PMC11333854/
- GitHub：https://github.com/mgy520/spatialQC
- 教程文档：https://mgy520.github.io/SpatialQC
- Zenodo：https://doi.org/10.5281/zenodo.12634669
