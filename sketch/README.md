# 空间转录组为什么要做 Sketch？一篇 NAR 基准测试告诉你

> 微信稿 | 解读 *Nucleic Acids Research*, 2026, gkag434  
> Gingerich I K, Goods B A, Frost H R. Benchmarking sketching methods on spatial transcriptomics data  
> 原文：[DOI: 10.1093/nar/gkag434](https://academic.oup.com/nar/article/54/9/gkag434/8675578) | [bioRxiv](https://doi.org/10.1101/2025.08.26.672376) | [代码](https://github.com/gingerii/Benchmarking-sketching-for-spatial-transcriptomics)  
> 关联实践：[Seurat Visium HD vignette](https://satijalab.org/seurat/articles/visiumhd_analysis_vignette)

---

## 开篇

Visium HD 一张片 **8 μm bin** 动辄 **30 万～40 万** 列；Stereo-seq、MERFISH atlas 更是百万级。全量跑 PCA、建图、Louvain、去卷积——笔记本扛不住，服务器也慢。

于是 **Sketch（智能子抽样）** 成了大样本空间转录组的标配：先抽一小撮「有代表性」的 spot/bin，在子集上算聚类或注释，再投影回全量。

但 Dartmouth 团队在 *NAR* 上问了一个更尖锐的问题：**从 scRNA-seq 搬过来的 Sketch 方法，直接用在 ST 上，会不会把组织「抽歪」？**

答案：**会。** 而且抽歪的方式，恰恰和空间数据最在乎的「组织形态」有关。

---

## 一、Sketch 是什么？解决什么问题？

**Sketching = 智能子抽样（intelligent sub-sampling）**：从 N 万个空间位点里选出 n 个（通常 n ≪ N），让子集在后续分析里**尽量代表全量**。

| 痛点 | Sketch 的作用 |
|------|----------------|
| 矩阵太大，PCA / 聚类 / 去卷积太慢 | 在子集上算，再映射全量 |
| 稀疏 bin 多，全量图噪声大 | 子集更稳、更快收敛 |
| 稀有细胞类型 bin 数少 | 好的 sketch 应**保留稀有群** |

单细胞领域早有成熟方案：**Leverage score**（Seurat v5）、**Geosketch**（Hie et al., 2019）、**scSampler**（maximin 多样性）等——目标都是在**表达空间**里均匀覆盖异质性。

gkag434 的核心论点：**ST 多了一个维度——物理坐标。** 只优化表达空间，会牺牲组织覆盖；只优化空间，又会漏掉转录极端态。**空间转录组需要「空间 + 转录」联合的 Sketch 目标。**

---

## 二、规模有多大？为什么现在才成为问题？

| 平台 / 数据 | 量级（约） |
|-------------|------------|
| 经典 Visium | ~5,000 spot/片 |
| Visium HD（8 μm） | **30 万～40 万 bin/片** |
| Stereo-seq、大 MERFISH atlas | **10 万～百万+** 细胞/位点 |

Seurat Visium HD 教程里，小鼠脑 8 μm 数据约 **39 万 bin**；在 5 万 sketch 上跑 Louvain 约 **27 秒**，全量 BANKSY 聚类同类操作约 **201 秒**——这还是「能放进内存」的情况。

高通量 ST 已经把瓶颈从「测序」挪到 **常规生信分析**。Sketch 不是炫技，而是让聚类、降维、去卷积在 atlas 尺度上**可 routinely 运行**的前提。

---

## 三、gkag434 做了什么？

作者系统 benchmark 了 **4 类 sketch 策略** × **3 种输入表示**，在多个真实 ST 数据集（小鼠卵巢、MERFISH 脑、人乳腺癌、肺）和模拟数据上评估。

### 3.1 四种 Sketch 方法

| 方法 | 思路 | 来源 |
|------|------|------|
| **Uniform sampling** | 随机均匀抽样 | 基线 |
| **Leverage-score sampling** | 按表达空间「杠杆分数」选信息量大的点 | scRNA-seq / Seurat v5 |
| **Geosketch** | minimax / Hausdorff，表达空间几何覆盖 | Hie et al., *Cell Systems* 2019 |
| **scSampler** | maximin，多样性子抽样 | Song et al., *Bioinformatics* 2022 |

### 3.2 三种输入表示

| 输入 | 含义 |
|------|------|
| **PCA embedding** | 纯转录组降维空间 |
| **Spatial coordinates** | 纯物理坐标 |
| **Spatially smoothed embedding** | PCA 经**空间权重矩阵平滑**后的表示 |

### 3.3 评价指标

- **Robust Hausdorff distance** — 子集对全量空间分布的覆盖  
- **聚类稳定性（ARI）** — sketch 前后聚类是否一致  
- **PCA loading drift** — 降维结构是否被扭曲  
- **Local cell-type MSE** — 局部细胞类型比例是否保真  

---

## 四、关键发现：scRNA 的 Sketch 用在 ST 上会「抽歪」

### ① 只用表达（PCA / Leverage / Geosketch）

- **优点**：抓住全局转录异质性，稀有转录状态不易漏  
- **缺点**：**过度抽样高变异区域**（肿瘤边缘、层状边界、免疫浸润带），**欠抽样同质区域**（大片均匀皮质、基质）  
- **后果**：组织架构在 sketch 里变形——空间图看起来「该平的地方不平」

### ② 只用坐标（uniform on xy）

- **优点**：组织覆盖均匀，不遗漏物理区域  
- **缺点**：**漏掉转录极端态**——稀有细胞类型、高表达 outlier 可能被忽略  
- **后果**：聚类稳定，但生物学上「该有的稀有群没了」

### ③ 论文推荐：**Spatially smoothed leverage scores**

做法简述：

1. 对表达矩阵做 randomized SVD，得到 PCA 基  
2. 用**空间权重矩阵**对基向量/embedding 做平滑（邻域加权）  
3. 在平滑后的空间上计算 leverage score 并抽样  

**效果**：在「保留稀有细胞状态」和「维持均匀组织覆盖」之间取得平衡，还能减轻组织边缘效应。

> 论文结论：在 Hausdorff、ARI、PCA drift、local cell-type MSE 上，**spatially smoothed leverage scores 与现有方法相比持平或更优**。

---

## 五、和 Seurat Visium HD 教程怎么对上号？

[Seurat Visium HD vignette](https://satijalab.org/seurat/articles/visiumhd_analysis_vignette) 里的流程：

```r
object <- SketchData(object, ncells = 50000, method = "LeverageScore", sketched.assay = "sketch")
# 在 sketch 上聚类
FindClusters(object, cluster.name = "seurat_cluster.sketched", ...)
# 投影回全量
object <- ProjectData(object, ..., refdata = list(seurat_cluster.projected = "seurat_cluster.sketched"))
```

你会看到两张图：

| 图 | 含义 |
|----|------|
| **Sketched clustering (50,000 cells)** | 仅在 5 万 sketch bin 上学到的聚类 / UMAP |
| **Projected clustering (full dataset)** | 把标签和降维**投影到全部 bin** |

**为什么要 Sketch？**（Seurat 官方 + gkag434 合读）

1. **算力**：数十万 bin 全量 Louvain / UMAP 慢  
2. **稀有群**：Leverage 抽样偏向信息量大的点，利于发现**空间局限的稀有 cluster**（如特定皮层亚层）  
3. **可扩展**：同一套路用于 RCTD 去卷积——先在 sketch 上跑，再 `ProjectData` 回全量  

**gkag434 的提醒**：Seurat 默认的 **LeverageScore 针对表达空间优化**；在 ST 上可能带来**空间抽样偏倚**。若你关心组织全覆盖（尤其大片同质区 + 稀疏稀有群并存），应关注论文提出的 **spatially smoothed leverage**——或在分析中**目视检查 sketch 的空间分布是否均匀**。

---

## 六、实操建议：做 Sketch 之前想清楚三件事

### 1. 你的下游需要什么？

| 下游任务 | Sketch 侧重点 |
|----------|----------------|
| 无监督聚类 / UMAP | 聚类稳定性（ARI）、稀有群是否保留 |
| 空间域（BANKSY 等） | 空间覆盖（Hausdorff）、边界是否完整 |
| RCTD / 去卷积 | 局部 cell-type 比例（local MSE） |
| 与 scRNA 整合 | 转录极端态不能丢 |

### 2. 抽多少？抽完必须「投影回去」

- Visium HD 常见 **n = 50,000**（约为全量 10%～15%）  
- **不要**只分析 sketch 子集就下结论——空间发表图应用 **projected** 全量标签  
- 对比：随机抽 5 万 vs Leverage 5 万 vs spatially smoothed 5 万，看空间 overlay 差异  

### 3. Sketch ≠ Cell Segmentation

| 概念 | 解决什么 |
|------|----------|
| **Sketch** | 位点太多，**抽哪些 bin/spot 代表全量** |
| **Cell segmentation** | bin 不等于细胞，**哪些 bin 属于同一个 cell** |

Visium HD 上二者常先后出现：8 μm bin 全量 → Sketch 聚类 →（可选）2 μm + 分割得 cell 矩阵。别混为一谈。

---

## 七、方法选型速查（基于 gkag434）

| 场景 | 建议 |
|------|------|
| 快速探索、无特殊要求 | Seurat `LeverageScore` sketch + `ProjectData`（生态成熟） |
| 组织同质区很大、担心「只抽边界」 | 关注 **spatially smoothed leverage**；或对比 uniform / Geosketch |
| 更看重空间全覆盖 | 坐标分层抽样 + 表达 leverage 混合（论文动机方向） |
| 已有 scRNA 工作流、数据是 MERFISH / CosMx 单细胞级 | 同样存在规模问题，**不能用纯表达 sketch 默认值 blindly** |
| 全量能跑且不太大（经典 Visium ~5k） | **不必 sketch**——杀鸡不用牛刀 |

---

## 八、论文信息卡片

| 项目 | 内容 |
|------|------|
| **标题** | Benchmarking sketching methods on spatial transcriptomics data |
| **期刊** | *Nucleic Acids Research*, 2026, 54(9): gkag434 |
| **作者** | Ian K. Gingerich, Brittany A. Goods, H. Robert Frost（Dartmouth College） |
| **预印本** | bioRxiv 2025.08.26.672376 |
| **开源** | [GitHub: Benchmarking-sketching-for-spatial-transcriptomics](https://github.com/gingerii/Benchmarking-sketching-for-spatial-transcriptomics) |
| **许可** | CC BY-NC 4.0 |

**一句话 takeaway**：ST 的 Sketch 不能只看转录异质性，还必须看组织在空间上是否被**均匀、无偏**地代表——**spatially smoothed leverage score** 是目前 benchmark 里较均衡的选择；用 Seurat 分析 Visium HD 时，理解 `SketchData` → `ProjectData` 是在「子集学习、全量展示」，并留意纯 leverage 的空间偏倚。

---

## 写在最后

空间转录组正在从「一片几千 spot」变成「一片几十万 bin」——**Sketch 是大样本 ST 分析的基础设施**，不是可有可无的优化。

gkag434 的价值在于：它用系统 benchmark 说明，**从 scRNA-seq 继承的抽样逻辑，默认会扭曲 ST 的组织架构**。这对所有做 Visium HD、Stereo-seq、MERFISH atlas 的人都有直接意义：

1. **要做 Sketch**——否则算不动、稀有群难找  
2. **要会做对 Sketch**——表达空间与物理空间要 jointly 考虑  
3. **Sketch 结果要投影回全量**——子集聚类只是中间步，不是终点  

如果你正在跑 Seurat Visium HD pipeline，建议在 `SpatialDimPlot` 之外，加一张 **「sketch 位点的空间分布图」**：若点全堆在组织边缘或高变异区，就该换方法或调 spatial smoothing 了。

---

## 参考文献

1. Gingerich I K, Goods B A, Frost H R. Benchmarking sketching methods on spatial transcriptomics data. *Nucleic Acids Research*, 2026, 54(9): gkag434. https://doi.org/10.1093/nar/gkag434  
2. Hao Y, et al. Dictionary learning for integrative, multimodal and scalable single-cell analysis. *Nature Biotechnology*, 2024. (Seurat v5 sketch)  
3. Hie B, et al. Geometric sketching compactly summarizes the single-cell transcriptomic landscape. *Cell Systems*, 2019.  
4. Satija Lab. Analysis, visualization, and integration of Visium HD spatial datasets with Seurat. https://satijalab.org/seurat/articles/visiumhd_analysis_vignette  
5. Song D, et al. scSampler: fast diversity-preserving subsampling of large-scale single-cell transcriptomic data. *Bioinformatics*, 2022.
