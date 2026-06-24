# 会做单细胞还不够：空间转录组第一步，为什么要找 SVG？

> **摘要**：你已经会找 HVG、做聚类、做注释了。进入空间转录组后，第一个关键步骤变成了 **SVG（Spatially Variable Genes，空间可变基因）**。这篇帮你从单细胞视角快速搞懂：SVG 是什么、和 HVG 有何不同、34 种方法怎么选。

---

如果你做过单细胞 RNA-seq，下面这些步骤一定不陌生：

```
质控 → 归一化 → 找 HVG → 降维 → 聚类 → 注释
```

其中 **HVG（Highly Variable Genes，高可变基因）** 是降维和聚类的「燃料」——找的是在**细胞与细胞之间**表达差异大的基因。

但当你第一次拿到 Visium、Xenium 或 Illumina Spatial 的数据，照搬单细胞流程，很容易踩第一个坑：

> **HVG 找出来的基因，不一定有空间模式；有空间模式的基因，也不一定是传统意义上的 HVG。**

这就是 **SVG** 要解决的问题。

---

## 一、从 HVG 到 SVG：多出来的那一维

| | **HVG（单细胞）** | **SVG（空间转录组）** |
|---|---|---|
| **问的问题** | 哪些基因在细胞间差异大？ | 哪些基因在**空间上**有非随机分布？ |
| **利用的信息** | 表达矩阵 | 表达矩阵 + **坐标** |
| **典型用途** | 降维、聚类、批次校正 | 特征筛选、空间域发现、机制解读 |
| **可能遗漏** | 有空间梯度但细胞间差异不大的基因 | 细胞类型 marker，但本身无显著空间模式 |

举个例子：某个基因在整张切片上均匀高表达，细胞间差异很大——它是 **HVG**，但不是 **SVG**。

反过来，某个基因只在皮层某一层呈条带状表达，整体细胞间差异未必最大——它可能是 **SVG**，却未必排在 HVG 前列。

[Single-cell best practices 教程](https://www.sc-best-practices.org/spatial/spatially_variable_genes.html) 用一句话概括：

> **SVG 是一种「正交」的特征选择方式——选的是在空间中可变的基因，而不是在观测之间可变的基因。**

---

## 二、SVG 为什么重要？

Adhikari 等人在 2024 年的综述中指出，SVG 检测是空间转录组**下游分析的初始关键步骤**，许多后续任务都依赖它：

- 空间聚类 / **空间域（spatial domain）** 识别
- 空间差异表达
- 细胞通讯的空间背景解读
- 与 H&E 形态学对照，理解组织结构

空间变异可能来自：

- 不同**细胞类型**的空间排布
- 同一细胞类型内的**功能状态**梯度
- **细胞-细胞通讯**或微环境效应

找 SVG，本质上是在问：**哪些基因的表达，真的「跟着组织走」？**

---

## 三、先搞懂：SVG 其实有三种（不是只有一种）

这是 Yan 等人 2025 年发表在 *Nature Communications* 上综述的核心贡献——他们梳理了 **34 种** SVG 检测方法，并指出业界长期存在一个根本问题：

> **不同方法对「空间可变」的定义不同，结果自然不可比。**

因此，他们把 SVG 分为 **三类**：

### 1. Overall SVGs（整体空间可变基因）

- **定义**：在整个组织层面，基因表达呈现非随机的空间模式
- **用途**：为下游分析（空间域识别、功能模块）筛选**信息量大**的特征基因
- **类比单细胞**：最接近「全局 HVG」，但加了空间约束

### 2. Cell-type-specific SVGs（细胞类型特异性 SVG）

- **定义**：在**某一种细胞类型内部**，基因表达仍随空间位置变化
- **用途**：发现同一细胞类型的**空间亚状态**（如肿瘤边缘 vs 核心的巨噬细胞）
- **需要**：已知或推断的细胞类型信息

### 3. Spatial-domain-marker SVGs（空间域 marker 基因）

- **定义**：在**已划定的空间域内**高表达的基因
- **用途**：解释某个空间区域的分子特征
- **需要**：先做空间域划分，再找 marker

**对单细胞用户的提醒**：

> 不要问「哪个 SVG 方法最好」，要先问「我要找的是哪一种 SVG」。

---

## 四、主流方法怎么分？（不用记 34 个名字）

Yan 等人按统计原理把方法分为三大类；Adhikari 等人则按是否建模、输入数据类型来分。结合起来，你可以这样理解：

### 路线 A：空间自相关（最快上手）

代表：**Moran's I**（Squidpy 内置）

- **原理**：相邻 spot/细胞的表达是否更相似？
- **优点**：计算快，Scanpy/Squidpy 生态友好，教程多
- **代码量级**（来自 [sc-best-practices](https://www.sc-best-practices.org/spatial/spatially_variable_genes.html)）：

```python
import squidpy as sq

sq.gr.spatial_neighbors(adata)
sq.gr.spatial_autocorr(adata, mode="moran")
# 结果在 adata.uns["moranI"]
```

- **局限**：主要面向 **overall SVG**；对复杂非线性模式可能不够敏感

### 路线 B：模型驱动（Gaussian Process 等）

代表：**SpatialDE**、**SpatialDE2**、**SPARK** / **SPARK-X**

| 方法 | 核心思路 | 输入数据 |
|------|----------|----------|
| **SpatialDE** | 高斯过程回归，分解空间/非空间方差 | 需归一化、方差稳定 |
| **SPARK** | 拟泊松/高斯空间模型，多 kernel 候选 | 原始 count 或归一化 |
| **SPARK-X** | 非参数，检验表达与坐标是否独立 | count，可扩展 |

- **优点**：统计框架完整，有 p 值 / q 值
- **局限**：SpatialDE 在全基因上较慢；kernel 选择影响结果（Adhikari 综述特别强调）

### 路线 C：图/扩散/无模型

代表：**Sepal**（扩散模拟）、**scGCO**（图割）、**BinSpect**、**MERINGUE**

- 不依赖特定分布假设，或利用空间图结构
- Chen X 等人 2025 年 benchmark 中，**BinSpect**、**SPARK-X** 在多种指标上表现突出；Chen C 等人 2024 年则指出 **SPARK-X** 在速度、稀疏稳健性和模拟 TPR/FDR 上综合较优

---

## 五、benchmark 告诉我们什么？（两篇 Chen 文，别搞混）

空间 SVG 领域有两篇常被引用的 benchmark，作者都叫 Chen，但团队、年份、方法范围完全不同。**做分析前建议先确认引用的是哪一篇。**

| | **Chen C et al., 2024** | **Chen X et al., 2025** |
|---|---|---|
| **期刊** | *Genome Biology* 25(1): 18 | *Bioinformatics* 41(4): btaf131 |
| **方法数** | 8 种 | 15 种 |
| **数据集** | 31 个真实 + 模拟 | 30 个模拟 + 74 个真实 |
| **开放获取** | ✅ [全文免费](https://doi.org/10.1186/s13059-023-03145-y) | 视订阅情况 |
| **侧重点** | 方法一致性、FDR 校准、稀疏稳健性 | 准确性、分辨率分层、下游聚类 |

下面分别解读。

---

### 5.1 Chen C 等（Genome Biology, 2024）：8 种方法的「真相检验」

**文献**：Chen C, Kim H J, Yang P. Evaluating spatially variable gene detection methods for spatial transcriptomics data. *Genome Biology*, 2024, 25(1): 18.  
DOI：[10.1186/s13059-023-03145-y](https://doi.org/10.1186/s13059-023-03145-y)（开放获取）

**评估的 8 种方法**：

- **Giotto** 两条路线：KM（k-means 二值化）与 rank（排名阈值二值化）
- **Moran's I**（Seurat 内置）
- **MERINGUE**（Voronoi 邻域 + Moran's I）
- **nnSVG**（最近邻高斯过程）
- **SOMDE**（自组织映射 + 高斯核）
- **SPARK-X**（非参数协方差检验）
- **SpatialDE**（高斯过程混合模型）

**数据覆盖**：31 个真实数据集，横跨 **Visium、ST、Slide-seq、MERFISH、Stereo-seq** 等 11 种技术；另用 scDesign3 构建带 ground truth 的模拟数据（约 10% 基因为真 SVG）。

#### 核心发现 1：排名「有点像」，显著基因「几乎不重叠」

各方法对基因的**相对排名**有一定 Spearman 相关（多数数据集 >0.5），但按 **FDR ≤ 0.05** 筛显著 SVG 时，**8 种方法的交集极小**——不少数据集里，几乎没有基因被全部 8 种方法同时检出。

> **对实操的启示**：不同工具给出的「显著 SVG 列表」差异可以非常大。**不要只信一份 p 值清单**，多方法交叉 + 空间图验证是刚需，而非可选项。

方法之间大致形成两个「同盟」：

- **同盟 A**：Giotto KM ↔ Giotto rank（彼此最一致）
- **同盟 B**：MERINGUE ↔ Moran's I ↔ nnSVG
- **相对孤立**：SOMDE、SPARK-X、SpatialDE 与其他方法一致性最低；**SpatialDE 与多数方法的相关性最弱**，在 Visium 上相对好一些

#### 核心发现 2：多数方法有「高表达偏好」（类似 HVG 的坑）

几乎所有方法检出的 SVG，其表达量都偏高；**SPARK-X 的偏好最强**（与表达量 Spearman 相关约 **0.8**）。这与单细胞里 HVG 检测的 expression bias 非常像。

> **对实操的启示**：转录因子等低表达但空间上有意义的基因，容易被当前 SVG 工具漏掉。解读结果时要有心理预期，必要时单独关注低表达候选基因。

#### 核心发现 3：对 spot 稀疏和数据扰动，方法表现分化明显

随机下采样 **80% spot** 后：

| 维度 | 相对稳健 | 较敏感（排名波动大） |
|------|----------|----------------------|
| **排名稳定性** | SPARK-X、SpatialDE、Moran's I | nnSVG、SOMDE、MERINGUE、Giotto |
| **假阳性控制**（下采样后新增显著基因比例） | SPARK-X、SOMDE、SpatialDE | Giotto、MERINGUE 等 |

另外，当随机去掉 **50% 基因** 再跑 SVG 时，多数方法的排名会发生变化——说明当前工具在一定程度上把基因间的**相互依赖**写进了结果里，预处理（基因过滤）会实质影响输出。

#### 核心发现 4：模拟数据上，FDR 校准并不都靠谱

在已知 ground truth 的模拟数据上：

- **TPR 高且 FDR 控制好**：**SPARK-X、SOMDE、nnSVG、SpatialDE**
- **TPR 高但 FDR 严重偏高**：**Moran's I、Giotto（KM / rank）**——报告的 adjusted p 值**不能**当作真实 FDR 来用

作者因此在 Discussion 中明确警告：

> 统计显著性的估计仍然困难；**不宜过度依赖部分工具的 p 值来做生物学结论**，尤其是 Moran's I 和 Giotto 系列。

#### 核心发现 5：下游空间域聚类，SVG 数量有「甜点区」

以小鼠 E9.5 胚胎数据为例，用各方法检出的 top SVG 做 PCA + 聚类（对比 BayesSpace、SpaGCN、k-means 等），**约 900–1100 个** top SVG 时聚类与已知组织域的一致性（ARI、NMI 等）最好；太少信息不够，太多则引入噪声。

#### 核心发现 6：算力与可扩展性

- **最快、最易扩展**：**SPARK-X**（Visium 全基因 <1 分钟量级）
- **最慢**：**SpatialDE**（大样本全基因可达数小时）

---

### 5.2 Chen X 等（Bioinformatics, 2025）：15 种方法的扩展 benchmark

Chen X 等人在 *Bioinformatics*（2025, btaf131）将评估扩展到 **15 种**方法（含 BinSpect、SPARK、dCor、HSIC 等 Chen C 文未覆盖的工具），使用 **30 个模拟 + 74 个真实**数据集，评估维度包括：

- 检测准确性（AUPR、AUROC）
- 统计有效性（FDR 控制）
- 下游聚类效果
- 稳定性与可扩展性

#### 几条实用结论（面向选方法的人）

**1. 没有「全能冠军」，但有「常胜选手」**

综合表现较好的包括：**BinSpect、SPARK、SpatialDE、SPARK-X、dCor** 等，但具体排名因评价标准和数据集而异。

**2. 数据分辨率很重要**

| 分辨率 | 特点 | 方法选择提示 |
|--------|------|--------------|
| **低分辨率**（如 Visium >50 μm） | 空间模式较平滑 | 空间感知聚类（如 BayesSpace）+ SVG 组合效果更好 |
| **高分辨率**（<20 μm，如 Slide-seq、Stereo-seq） | 更稀疏、更精细 | BinSpect 较稳健；Moran's I 类方法（MERINGUE）在高分辨率稀疏数据上可能偏弱 |
| **中分辨率** | 介于两者之间 | SOMDE 等合并邻近位点的方法可能更有优势 |

**3. 相关类方法 FDR 可能偏松**

HSIC、dCor、BinSpect、SPARK-X 等基于「表达-坐标相关性」的方法，报告的 SVG 比例可能偏高（平均 >35% 基因），使用时注意 **多重检验校正** 和 **人工验证**。

**4. 下游任务决定 SVG 选择**

- 若目标是 **空间域聚类**：低分辨率数据优先考虑 SVG + **BayesSpace**
- 若目标是 **高分辨率精细分域**：SVG + 传统 scRNA-seq 聚类（Louvain/Leiden）有时反而更好

#### 两篇 Chen 文合在一起，怎么选？

| 你的场景 | 更倾向参考 |
|----------|------------|
| Visium / 10x 入门，要快速筛 + 稳健 | **SPARK-X**（两篇文均表现突出；2024 文强调速度与稀疏稳健性） |
| Squidpy / Scanpy 生态，先跑通流程 | **Moran's I**（快、教程多；但 **p 值需谨慎**，建议配合可视化） |
| 需要统计框架完整、模拟数据 FDR 较准 | **SpatialDE、SOMDE、nnSVG** |
| 数据很稀疏、spot 数波动大 | 优先 **SPARK-X / SpatialDE / SOMDE**；慎用对下采样敏感的 Giotto / MERINGUE |
| 做空间域聚类 | top SVG 取 **~1000 个** 左右试探（Chen C 2024）；低分辨率可叠加 **BayesSpace**（Chen X 2025） |

---

## 六、给「会单细胞、刚入门空间」的 5 条实操建议

这是综合 [sc-best-practices 教程](https://www.sc-best-practices.org/spatial/spatially_variable_genes.html)、Adhikari 2024 综述，以及 Chen C 2024、Chen X 2025、Yan 2025 论文的建议：

### 1. 先明确你要哪一种 SVG

- 探索性分析 → **overall SVG**
- 已有细胞注释 → 考虑 **cell-type-specific SVG**
- 已有空间域 → 找 **domain marker**

### 2. 不要只跑一种方法，更不要只信 p 值

Chen C 等（2024）在 31 个数据集上证实：各方法排名有一定相关，但 **FDR ≤ 0.05 的显著基因重叠极低**；Moran's I、Giotto 的 adjusted p 值在模拟数据中**不能准确反映真实 FDR**。

sc-best-practices 也明确建议：**用多种方法交叉验证**，并在组织图像上 **肉眼看空间表达图**。

### 3. 入门推荐组合

```
Squidpy Moran's I（快速筛） + SpatialDE 或 SPARK-X（统计检验） + 空间表达可视化
```

### 4. SVG 结果一定要「看图」

无论 p 值多显著，都要用 `sq.pl.spatial_scatter()` 或类似函数，把 top SVG 叠加到 H&E / 组织图像上。

**没有空间图验证的 SVG 列表，不要直接进下游。**

### 5. 区分「SVG」和「marker 基因」

一个基因可以是某 cluster 的 marker，同时有清晰空间模式；也可能只是 cluster marker，空间上并无特殊分布。Yan 等人的三分法，就是在提醒你：**生物学问题不同，基因清单的含义也不同。**

---

## 七、一张图串起来：从单细胞到空间的第一步

```
单细胞思路                    空间转录组思路
─────────                    ─────────────
HVG（细胞间变异）      →      SVG（空间变异）
聚类（细胞类型）       →      空间域（组织区域）
Marker（细胞类型）     →      SVG / Domain marker（位置相关）
Cell-cell comm         →      空间邻域 + 配体受体
```

你已有的单细胞技能（Scanpy、差异分析、可视化）**大部分可以复用**；真正新增的是：

- **空间邻域图**（spatial neighbors）
- **坐标信息**（`adata.obsm["spatial"]`）
- **组织图像**（`adata.uns["spatial"]` 或 OME-TIFF）

---

## 八、写在最后

SVG 检测看起来只是「多跑一个基因列表」，实则是空间转录组从「单细胞思维」转向「组织思维」的第一道门。

方法从 Moran's I 到 34 种算法，不必一次全懂。记住三句话就够：

1. **HVG ≠ SVG**，空间信息是新的维度  
2. **先定 SVG 类型，再选方法**（overall / cell-type-specific / domain-marker）  
3. **多种方法 + 空间可视化**，比任何一个 p 值都可靠（Chen C 2024 用 31 个数据集证明了这一点）  

---

## 参考文献

1. Palla G, et al. Squidpy: a scalable framework for spatial omics analysis. *Nature Methods*, 2022.  
   教程：[Spatially variable genes — Single-cell best practices](https://www.sc-best-practices.org/spatial/spatially_variable_genes.html)

2. Adhikari SD, Yang J, Wang J, et al. Recent advances in spatially variable gene detection in spatial transcriptomics. *Computational and Structural Biotechnology Journal*, 2024, 23: 883-891.  
   [PMC 全文](https://pmc.ncbi.nlm.nih.gov/articles/PMC10869304/)

3. Chen C, Kim H J, Yang P. Evaluating spatially variable gene detection methods for spatial transcriptomics data. *Genome Biology*, 2024, 25(1): 18.  
   [开放获取全文](https://doi.org/10.1186/s13059-023-03145-y)

4. Chen X, Ran Q, Tang J, et al. Benchmarking algorithms for spatially variable gene identification in spatial transcriptomics. *Bioinformatics*, 2025, 41(4): btaf131.

5. Yan G, Hua SH, Li JJ. Categorization of 34 computational methods to detect spatially variable genes from spatially resolved transcriptomics data. *Nature Communications*, 2025, 16: 1141.  
   [Nature 全文](https://www.nature.com/articles/s41467-025-56080-w)

---
