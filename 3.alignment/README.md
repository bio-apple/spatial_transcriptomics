# 空间转录组多切片对齐与整合

> **整合解读** | 两篇互补文献  
> **综述地图**：Khan M A, Arslanturk S, Draghici S. *Nucleic Acids Research*, 2025, 53(12): gkaf536 — [DOI](https://doi.org/10.1093/nar/gkaf536)  
> **系统评测**：Yan Y, Gu T, Sun C, et al. *Nature Computational Science*, 2026 — [DOI](https://doi.org/10.1038/s43588-026-00977-z)  
> **开源工具**： [SABench](https://github.com/Yunzhi-Yan/SABench) | [Zenodo](https://doi.org/10.5281/zenodo.18605715)

---

## 摘要

单张 Visium 切片约 5000 个 spot，Visium HD / Xenium 一张可达 **20 万～50 万** 位点。要理解完整组织，通常需要**多张连续切片**叠成 3D 视图，或把**不同实验、不同平台**的切片对齐到同一 **CCS（Common Coordinate System，公共坐标系）**。

这就是空间转录组里的 **alignment（对齐）** 与 **integration（整合）**——保留基因表达模式与空间关系，同时消除切片间形变与批次差异。

本文整合两篇关键文献：

1. **NAR gkaf536 综述** — 24 种工具、三大流派、7 步通用 pipeline、任务分类框架  
2. **SABench 系统评测** — 11 种主流方法在 295 个对齐任务上的精度、效率、鲁棒性与困难场景实测

---

## 一、为什么这件事很重要？

ST 数据来自**二维切片**，每张只是组织的一「层」。多切片整合才能：

- 重建 **3D 基因表达图谱**（组织 atlas）
- 追踪 **发育时间轴**（胚胎不同时间点）
- 比较 **疾病 vs 健康** 的空间表达差异
- 提升低表达区域的 **基因覆盖度**
- 支撑空间域识别、轨迹分析、细胞类型注释

### 核心挑战

| 挑战 | 具体表现 |
|------|----------|
| **组织异质性** | 不同切片区域、发育阶段结构差异大 |
| **空间 warping** | 切片、固定、成像导致坐标变形 |
| **批次效应** | 实验协议、平台分辨率不同 |
| **高维稀疏** | 2 万+ 基因 × 数十万 spot，计算昂贵 |
| **部分重叠** | 非连续切片、跨数据集时区域仅部分对应 |

### 问题定义

**输入**：多张切片的基因表达矩阵 + 空间坐标（± H&E 组织学图像）

**输出**：对齐并整合到 CCS 的统一表示

每张切片 \(X_i\) 通过变换 \(f_i: X_i \rightarrow Y\) 映射到公共空间，优化目标是最小化变换代价，同时保持生物学结构。

---

## 二、对齐任务的四个维度（gkaf536 Table 1）

| | **数据集内（Within）** | **跨数据集（Across）** |
|---|---|---|
| **同质（Homogeneous）** | 连续切片、同一区域、完全/最大重叠 | 不同样本/实验，但结构相似 |
| **异质（Heterogeneous）** | 非连续切片、不同区域或时间点 | 不同平台、分辨率、部分重叠 |

| 场景 | 任务类型 | 典型工具 |
|------|----------|----------|
| 连续 Visium 脑片 3D 重建 | 同质 + 数据集内 | PASTE / PASTE2 |
| 小鼠胚胎不同发育期 | 异质 + 数据集内 | STAligner / SpatiAlign |
| Visium + MERFISH 跨平台 | 异质 + 跨数据集 | SLAT / STalign / CAST |

---

## 三、通用 Pipeline 与工具地图（gkaf536）

### 3.1 七步通用流程

```
(A) 输入：基因表达 + 空间坐标 +（可选）H&E 图像
      ↓
(B) 预处理：归一化、log 变换、数据表示（矩阵 / 图 / 图像特征）
      ↓
(C) 降维与特征提取：PCA / UMAP / 图编码器
      ↓
(D) 对齐：低维嵌入间映射，优化 cost function
      ↓
(E) 整合：生成 CCS + 共享嵌入
      ↓
(F) 可视化：2D 空间图 + 3D 重建
      ↓
(G) 下游：组织 profiling、聚类、发育轨迹、疾病分析
```

### 3.2 三大流派（24 种工具）

#### 统计映射（SM）— 约 10 种

| 代表工具 | 核心技术 | 典型场景 |
|----------|----------|----------|
| **PASTE / PASTE2** | Fused Gromov-Wasserstein 最优传输 | 连续切片 pairwise / 中心切片整合 |
| **GPSA** | 高斯过程 + warping 函数 | 切片形变校正 |
| **PRECAST** | CAR 模型 + Potts 空间聚类 | 批次校正 + 空间域对齐 |
| **Splotch** | 贝叶斯层次模型 + HMC | 脊髓、嗅球 spatial profiling |
| **DeST-OT / ST-GEARS / OTVI** | 最优传输变体 | 发育时间点、非线性形变 |

**优势**：理论清晰，适合噪声和稀疏；**局限**：先验敏感，强非线性/跨批次可能不够鲁棒

#### 图像处理与配准（IPR）— 4 种

| 代表工具 | 特点 |
|----------|------|
| **STalign** | 微分同胚映射，跨 MERFISH / Visium |
| **STIM** | ImgLib2，3D 对齐框架 |
| **STUtility** | H&E superpixel + ICP landmark |
| **STaCker** | U-Net 深度学习，无 landmark |

**优势**：利用形态学结构；**局限**：需高质量 H&E，软组织 inter-sample 变异大时易失败

#### 图方法（GB）— 约 10 种

| 代表工具 | 特点 |
|----------|------|
| **STAligner** | 图注意力自编码器 + triplet loss |
| **SpatiAlign** | 对比学习 + 跨 batch 对齐 |
| **SLAT** | GCN + 对抗学习 + Wasserstein 图匹配 |
| **Graspot / SPIRAL** | GAT + 非平衡 OT / cluster-aware GW |
| **SPACEL / GraphST / STAIR** | 空间域识别 + 多切片整合 |

**gkaf536 作者建议**：异质/跨数据集场景优先关注 **autoencoder 系图方法**（STAligner、SLAT、SpatiAlign、Graspot）

### 3.3 常用 Benchmark 数据集

| 数据集 | 平台 | 用途 |
|--------|------|------|
| **DLPFC**（人前额叶皮层） | 10x Visium | 12 片连续切片，6 层皮层注释 |
| **Stereo-seq 小鼠胚胎** | Stereo-seq | 不同发育时间点 |
| **Slide-seq 海马** | Slide-seq | 连续切片 3D 重建 |
| **MERFISH 脑** | MERFISH | 跨冠状/矢状面对齐 |
| **Xenium 乳腺癌** | 10x Xenium | 病理注释验证 |

---

## 四、SABench 系统评测（Nat Comput Sci 2026）

gkaf536 指出领域**缺乏统一 benchmark**；复旦大学颜云智等发布 **SABench**，对 **11 种主流方法**在 **295 个对齐任务**上系统评测。

### 4.1 评测范围

**纳入的 11 种方法**

| 方法 | 对齐类型 | 特点 |
|------|----------|------|
| PASTE | 线性 | 经典多切片对齐 |
| PASTE2 | 线性 | 支持部分重叠切片 |
| STAligner | 混合 | 依赖 landmark 选择 |
| GPSA | 非线性 | 深度高斯过程 |
| SLAT | 混合 | 返回匹配点对 |
| STalign | 非线性 | 成对对齐；需单细胞分辨率 |
| CAST | 混合 | 支持跨平台；仅成对 |
| STAIR | 混合 | 对齐 + 整合 + 3D 重建 |
| SPACEL | 混合 | 利用区域信息，稳定性强 |
| Spateo | 混合 | 多切片表现优异 |
| SANTO | 混合 | 粗到细对齐；仅成对 |

**数据规模**：240 张切片，260 对真实 + 35 对模拟，**295 个对齐任务**；覆盖 Visium、Visium HD、MERFISH、Stereo-seq、Xenium、CosMx 等 **14 种**平台。

**六大评测维度**：精度、效率、鲁棒性、下游影响、困难场景、易用性（30+ 子项）

### 4.2 常规场景结果

| 平台类别 | 综合前三 |
|----------|----------|
| **NGS 类**（ST、Visium、Stereo-seq） | **PASTE2、SPACEL、Spateo** |
| **成像类**（MERFISH、BaristaSeq、STARmap） | **Spateo、STAIR、SPACEL** |

- Visium DLPFC：PASTE2、SPACEL、STAIR、CAST 领先  
- MERFISH 下丘脑：Spateo、PASTE2 最优  
- **平台类型显著影响方法排名**——没有跨平台通用冠军

### 4.3 下游任务影响

以 GraphST 3D 空间聚类为下游（DLPFC）：

- **12/15** 种对齐设置优于未对齐基线  
- **SPACEL、CAST、PASTE2** 下游最佳，ARI/NMI 等提升约 **0.3**

### 4.4 鲁棒性

| 扰动类型 | 最稳定 |
|----------|--------|
| 切片重叠比例变化 | **SPACEL** 全程高精度 |
| 初始旋转角度（0–360°） | **PASTE2、STAIR、SPACEL、Spateo** |
| 角度敏感（慎用） | PASTE、SLAT |

初始角度偏差在 **±30°** 内通常效果更好。

### 4.5 困难场景

| 场景 | 结论 |
|------|------|
| **跨平台** | 多数方法效果差；仅 MERFISH ↔ Xenium 略可接受；**当前最大短板** |
| **连续多切片（>10 张）** | **Spateo** 优势明显（16～129 片） |
| **大规模高分辨率** | Xenium 乳腺癌等：易内存溢出；需 **预对齐（PA）+ 降采样（DR）+ 调参** |

### 4.6 易用性与消融

- **最易用**：Spateo、PASTE 系列  
- **门槛较高**：GPSA、SANTO  
- **消融**：打乱空间坐标 → 仅 SLAT、PASTE_p0 可对齐；打乱基因表达 → **全部失败**  
- **结论**：方法最依赖 **基因表达相似性** + **相对空间位置**

---

## 五、整合选型指南

### 5.1 按场景快速选型（综述 + SABench 合并）

| 你的场景 | 优先考虑 |
|----------|----------|
| 连续 Visium 切片 3D 重建 | **PASTE / PASTE2** |
| NGS 类数据（Visium / Stereo-seq） | **PASTE2、SPACEL** |
| 成像类数据（MERFISH / Xenium） | **Spateo** |
| 不同发育时间点（异质） | **STAligner / SpatiAlign** |
| 有区域注释标签 | **SPACEL** |
| 跨平台对齐 | **CAST / SLAT / STAligner**（仍困难，无成熟方案） |
| 仅需匹配点对 | **SLAT** |
| 单细胞分辨率 + 密度结构 | **STalign** |
| 组织边界清晰 | **STAligner** |
| 有高质量 H&E | **STalign / STaCker** |
| 超大数据 / 内存溢出 | **预对齐 + 降采样 + Spateo 调参** |
| 需要不确定性估计 | **GPSA / Splotch** |

### 5.2 推荐工作流

```
1. 数据准备
   ├── 统一为 AnnData 格式
   ├── 质控（过滤细胞/基因）
   └── 粗预对齐（校正初始切片方向，±30° 内更佳）

2. 正式对齐
   ├── 按平台选择首选方法
   ├── 参数调优
   └── 内存溢出 → 降采样 / 进一步过滤

3. 对齐后检查
   ├── 视觉检查
   ├── 指标评估（PCC、SSIM、MI、ARI、区域重叠比等）
   └── 不满意 → 换方法重试

4. 下游分析
   └── 3D 聚类、空间域识别、细胞通讯等
```

### 5.3 特殊场景

| 场景 | 建议 |
|------|------|
| 内存溢出 | 降分辨率 → 对齐 → 恢复分辨率（PA+DR） |
| 对齐质量差 | 参数调优 → 换方法 → 手动 landmark |
| 跨平台 | CAST/STAligner 可尝试，但需降低预期 |
| 超大数据 | 预对齐 + 降采样 + 参数优化组合 |

---

## 六、下游应用（gkaf536）

| 应用方向 | 代表案例 |
|----------|----------|
| **组织 profiling** | PASTE 在 DLPFC 无监督恢复层 marker |
| **细胞类型/空间域聚类** | 整合后聚类优于纯 scRNA-seq 整合 |
| **发育分析** | STAligner 追踪小鼠胚胎器官比例变化 |
| **疾病进展** | ATAT 对齐 UC/CD 结肠切片 |
| **生物标志物** | GPSA 识别乳腺癌 PRSS23、CST4 |

---

## 七、尚未解决的问题与未来方向

| 问题 | 说明 |
|------|------|
| **可扩展性** | 百万级 spot 全自动对齐仍困难 |
| **跨平台对齐** | 生物学意义重大，SABench 证实普遍不足 |
| **H&E 利用不足** | 多数工具未充分做 3D 形态重建 |
| **软组织** | 脑以外 inter-sample 变异大的组织仍是难点 |
| **与 scRNA 整合混淆** | ST-spot 对齐 ≠ ST-scRNA 整合 |

**未来方向**：跨组学对齐、时空组学对齐、LLM 辅助对齐、标准化 benchmark（SABench 已迈出一步）

---

## 八、SABench 工具包

作者将评测框架封装为 **SABench**，支持：

- 快速计算评测指标，与 11 种方法直接对比  
- 用新数据 benchmark 新方法  
- 大规模数据的交互式粗对齐、网格降采样、分辨率恢复  
- 方法特异性参数调优策略  

**GitHub**: https://github.com/Yunzhi-Yan/SABench

---

## 总结对照

| 维度 | 核心结论 |
|------|----------|
| 综述（gkaf536） | 24 工具分 SM / IPR / GB 三流派；异质场景优先图+自编码器 |
| 评测（SABench） | **没有万能冠军**；平台与场景决定排名 |
| NGS 类 | PASTE2、SPACEL 精度领先 |
| 成像类 | Spateo 综合最优 |
| 鲁棒性 | PASTE2、SPACEL 最稳定 |
| 多切片 | Spateo 优势明显 |
| 跨平台 | 当前方法普遍不足 |
| 大数据 | 预对齐 + 降采样 + 参数优化 |
| 下游 | 好对齐显著提升 3D 空间聚类 |
| 依赖信息 | 基因表达相似性 + 相对空间位置 |

> **综述告诉你「有哪些路」；SABench 告诉你「哪条路在你这种数据上更稳」。**

---

## 参考文献

### 主文献

1. Khan M A, Arslanturk S, Draghici S. A comprehensive review of spatial transcriptomics data alignment and integration. *Nucleic Acids Research*, 2025, 53(12): gkaf536. https://doi.org/10.1093/nar/gkaf536

2. Yan Y, Gu T, Sun C, Zhang Y, Cui Y, Lin S, et al. Benchmarking alignment methods for spatial transcriptomics data. *Nature Computational Science*, 2026. https://doi.org/10.1038/s43588-026-00977-z

### 工具与数据

- SABench: https://github.com/Yunzhi-Yan/SABench  
- Zenodo: https://doi.org/10.5281/zenodo.18605715

### 主要对标方法

- PASTE: Zeira et al., *Nat Methods*, 2022  
- PASTE2: Liu et al., *Genome Res*, 2023  
- STAligner: Long et al., *Nat Commun*, 2022  
- Spateo: Qian et al., *Cell*, 2024  
- SPACEL: Li et al., *Nat Commun*, 2023  
- CAST: Fan et al., *Nat Methods*, 2023  
- SANTO: Li et al., *Nat Commun*, 2024  
- SLAT: Li et al., *Nat Methods*, 2023  
- STalign: Russell et al., *Nat Methods*, 2023
