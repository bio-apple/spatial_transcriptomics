# 多切片空间转录组怎么对齐？NAR 综述工具地图

> 论文解读 | Khan MA, Arslanturk S, Draghici S. *Nucleic Acids Research*, 2025, 53(12): gkaf536  
> 原文链接：[Oxford Academic](https://academic.oup.com/nar/article/53/12/gkaf536/8174767)  
> DOI：[10.1093/nar/gkaf536](https://doi.org/10.1093/nar/gkaf536)

---

## 摘要

单张 Visium 切片大约 5000 个 spot，Visium HD / Xenium 一张切片可达 **20 万～50 万** 细胞/位点。要理解完整组织，通常需要**多张连续切片**叠成 3D 视图，或把**不同实验、不同平台**的切片对齐到同一坐标系。

这就是空间转录组里的 **alignment（对齐）** 与 **integration（整合）**——把多张 2D 切片映射到 **CCS（Common Coordinate System，公共坐标系）**，同时保留基因表达模式与空间关系。

Dartmouth / Wayne State 团队这篇 NAR 综述，系统梳理了 **24 种** 多切片 ST 对齐与整合工具，给出通用 pipeline 和选型建议。

---

## 一、为什么这件事很重要？

ST 数据是**切片**测出来的——每张 2D 切片只是组织的一「层」。要回答以下问题，必须做多切片整合：

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

---

## 二、问题定义

**输入**：多张切片的基因表达矩阵 + 空间坐标（± H&E 组织学图像）

**输出**：对齐并整合到 **CCS** 的统一表示，保留 spot 间空间关系与表达模式

每张切片 \(X_i\) 通过变换函数 \(f_i: X_i \rightarrow Y\) 映射到公共空间，优化目标是最小化变换代价（loss），同时保持生物学结构。

---

## 三、对齐任务的四个维度

综述用 **Table 1** 把任务 scope 拆成 2×2 矩阵：

| | **数据集内（Within）** | **跨数据集（Across）** |
|---|---|---|
| **同质（Homogeneous）** | 连续切片、同一区域、完全/最大重叠 | 不同样本/实验，但结构相似 |
| **异质（Heterogeneous）** | 非连续切片、不同区域或时间点 | 不同平台、分辨率、部分重叠 |

### 选型提示

| 场景 | 任务类型 |
|------|----------|
| 连续 Visium 脑片 3D 重建 | 同质 + 数据集内（PASTE 经典场景） |
| 小鼠胚胎不同发育期 | 异质 + 数据集内（STAligner / SpatiAlign） |
| Visium + MERFISH 跨平台 | 异质 + 跨数据集（SLAT / STalign） |

---

## 四、通用 Pipeline（7 步）

```
(A) 输入：基因表达 + 空间坐标 +（可选）H&E 图像
      ↓
(B) 预处理：归一化、log 变换、数据表示（矩阵 / 图 / 图像特征）
      ↓
(C) 降维与特征提取：PCA / UMAP / 图编码器；空间自相关分析
      ↓
(D) 对齐：低维嵌入间映射，优化 cost function
      ↓
(E) 整合：生成 CCS + 共享嵌入
      ↓
(F) 可视化：2D 空间图 + 3D 重建
      ↓
(G) 下游：组织 profiling、聚类、发育轨迹、疾病分析
```

### 三种数据表示

| 表示方式 | 说明 |
|----------|------|
| **矩阵** | spot × 基因 + 坐标网格 |
| **图** | spot 为节点，空间邻接为边（neighborhood graph） |
| **图像** | H&E 特征、mask、superpixel |

---

## 五、24 种工具：三大流派

### 5.1 统计映射（SM）— 约 10 种

核心：PCA/UMAP 降维 + 统计模型建立切片间对应关系

| 代表工具 | 核心技术 | 典型场景 |
|----------|----------|----------|
| **PASTE / PASTE2** | Fused Gromov-Wasserstein 最优传输 | 连续切片 pairwise / 中心切片整合 |
| **GPSA** | 高斯过程 + warping 函数 | 切片形变校正、乳腺癌 |
| **PRECAST** | CAR 模型 + Potts 空间聚类 | 批次校正 + 空间域对齐 |
| **Splotch** | 贝叶斯层次模型 + HMC | 脊髓、嗅球 spatial profiling |
| **Eggplant** | GP + landmark 先验 | 跨个体 landmark 对齐 |
| **DeST-OT** | 最优传输 | 发育时间点对齐 |
| **ST-GEARS** | OT + Procrustes + 弹性场 | 非线性形变对齐 |
| **OTVI** | OT + Block Coordinate Descent | 多组学切片整合 |

**优势**：理论清晰，适合处理噪声和稀疏；PASTE 是领域奠基性工具

**局限**：先验假设敏感；对强非线性形变、跨数据集批次效应可能不够鲁棒

### 5.2 图像处理与配准（IPR）— 4 种

核心：利用 H&E 等组织学图像做配准

| 代表工具 | 特点 |
|----------|------|
| **STalign** | 微分同胚度量映射，跨 MERFISH / Visium |
| **STIM** | 基于 ImgLib2，3D 对齐框架 |
| **STUtility** | H&E superpixel + ICP landmark 配准 |
| **STaCker** | U-Net 深度学习，无 landmark 对齐 |

**优势**：利用形态学结构，脑等「原型结构」组织效果好

**局限**：需高质量 H&E；软组织 inter-sample 变异大时易失败

### 5.3 图方法（GB）— 约 10 种

核心：spot 构图 + 自编码器 / GNN / 对比学习 / 图匹配

| 代表工具 | 特点 |
|----------|------|
| **STAligner** | 图注意力自编码器 + triplet loss |
| **SpatiAlign** | 对比学习 + 跨 batch 对齐 |
| **SLAT** | GCN + 对抗学习 + Wasserstein 图匹配 |
| **Graspot** | GAT + 非平衡 OT |
| **SPIRAL** | GraphSAGE + cluster-aware Gromov-Wasserstein |
| **GraphST** | 借用 PASTE 中心对齐 |
| **SPACEL** | GCN + 对抗学习，多切片空间域 |
| **BiGATAE** | 注意力自编码器 |
| **ATAT** | CNN + triplet loss，H&E tile 对齐 |
| **MaskGraphene / STAIR** | 空间域识别 + 3D 映射 |

**优势**：处理非线性、高维、异质对齐；作者**重点推荐** autoencoder 系

**局限**：计算量大、超参敏感、训练数据需求高

---

## 六、Benchmark 常用数据集

| 数据集 | 平台 | 用途 |
|--------|------|------|
| **DLPFC**（人前额叶皮层） | 10x Visium | 12 片连续切片，6 层皮层注释 |
| **Stereo-seq 小鼠胚胎** | Stereo-seq | 不同发育时间点，异质对齐 |
| **Slide-seq 海马** | Slide-seq | 连续切片 3D 重建 |
| **MERFISH 脑** | MERFISH | 跨冠状/矢状面对齐 |
| **Xenium 乳腺癌** | 10x Xenium | 病理注释验证 |

### 评价指标

- **ARI**：聚类一致性
- **Hausdorff 距离**：空间覆盖
- **LISI**：批次混合
- **Landmark RMSE**：结构对齐精度
- **局部细胞类型 MSE**：微环境保真度

---

## 七、下游应用

| 应用方向 | 代表案例 |
|----------|----------|
| **组织 profiling** | PASTE 在 DLPFC 无监督恢复层 marker |
| **细胞类型/空间域聚类** | 整合后聚类优于纯 scRNA-seq 整合 |
| **发育分析** | STAligner 追踪小鼠胚胎器官比例变化 |
| **疾病进展** | ATAT 对齐 UC/CD 结肠切片 |
| **生物标志物** | GPSA 识别乳腺癌 PRSS23、CST4 |

---

## 八、工具选型速查

| 你的场景 | 优先考虑 |
|----------|----------|
| 连续 Visium 切片 3D 重建 | **PASTE / PASTE2** |
| 不同发育时间点（异质） | **STAligner / SpatiAlign** |
| 跨平台（Visium + Xenium / MERFISH） | **SLAT / STalign** |
| 有高质量 H&E，结构固定组织 | **STalign / STaCker** |
| 需要不确定性估计 | **GPSA / Splotch**（贝叶斯） |
| 大规模、多切片、强批次效应 | **STAligner / SLAT / Graspot** |

**作者总体建议**：优先关注 **autoencoder-based 图方法**（STAligner、SLAT、SpatiAlign、Graspot、BiGATAE）——在异质对齐、跨时间点、跨数据集场景下更鲁棒。

---

## 九、尚未解决的问题

- **可扩展性**：百万级 spot 的全自动对齐仍困难
- **H&E 利用不足**：多数工具未充分做 3D 形态重建
- **软组织对齐**：脑以外 inter-sample 变异大的组织仍是难点
- **缺乏统一 benchmark**：各工具自建评价框架
- **与 scRNA-seq 整合混淆**：ST-spot 对齐 ≠ ST-scRNA 整合

### 未来方向

- 多组学空间数据（蛋白/代谢）联合对齐
- 跨物种 spatial atlas
- 标准化 benchmark 数据集与评价框架
- 组织学 3D 重建与 ST 深度整合

---

## 十、写在最后

这篇综述的价值在于：

1. **正式定义**了 ST 多切片对齐/整合问题及其 scope
2. **24 工具 × 3 流派** 的系统性地图
3. 给出 **7 步通用 pipeline**
4. 明确推荐 **图 + 自编码器** 路线作为异质/跨数据集场景首选

如果你手上有多张 Visium / Xenium / MERFISH 切片，准备做 3D 重建或跨时间点比较——这篇 review 是值得收藏的方法学导航。

---

## 参考文献

1. Khan MA, Arslanturk S, Draghici S. A comprehensive review of spatial transcriptomics data alignment and integration. *Nucleic Acids Research*, 2025, 53(12): gkaf536. https://doi.org/10.1093/nar/gkaf536

2. Liu Y, Yang M, Deng Y, et al. (related prior reviews on ST alignment, cited in paper)

3. Hu J, Li X, et al. Computational methods for alignment and integration of spatially resolved transcriptomics data. *Comput Struct Biotechnol J*, 2024.

4. Zhang M, et al. PASTE: probabilistic alignment of spatial transcriptomics experiments. *Nature Methods*, 2022.

5. Zeira R, et al. Alignment and integration of spatial transcriptomics data. *Nature Methods*, 2022 (PASTE2).
