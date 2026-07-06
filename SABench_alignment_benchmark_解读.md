# SABench：空间转录组切片对齐方法系统评测解读

> 论文解读 | Yan Y, Gu T, Sun C, et al. *Nature Computational Science*, 2026  
> DOI: [10.1038/s43588-026-00977-z](https://doi.org/10.1038/s43588-026-00977-z)  
> 原文 PDF: `2026-Benchmarking alignment methods for spatial transcriptomics data.pdf`  
> 开源工具: [https://github.com/Yunzhi-Yan/SABench](https://github.com/Yunzhi-Yan/SABench)

---

## 一、研究背景

空间转录组（ST）技术能在组织原位同时捕获基因表达与空间坐标，但几乎所有平台都基于 **二维（2D）组织切片** 操作，而生物组织本质上是三维的。将多张切片**空间对齐（spatial alignment）** 到统一坐标系，是实现 3D 组织重建、跨切片整合和下游空间分析的关键计算步骤。

尽管 PASTE、STAligner、Spateo 等方法不断涌现，领域内一直缺乏：

- 专门针对**空间对齐**的系统 benchmark
- 覆盖**跨平台、连续切片、大规模数据**等真实困难场景的评价
- 面向用户的**方法选型指南**

复旦大学颜云智、顾天一、钱斌志等人在 *Nature Computational Science* 发表本研究，对 **11 种主流空间对齐方法** 进行系统评测，并发布开源 benchmark 工具包 **SABench**。

---

## 二、评测范围

### 2.1 纳入的 11 种方法

| 方法 | 对齐类型 | 特点 |
|------|----------|------|
| PASTE | 线性 | 经典多切片对齐 |
| PASTE2 | 线性 | 支持部分重叠切片 |
| STAligner | 混合 | 依赖 landmark 选择 |
| GPSA | 非线性 | 深度高斯过程 |
| SLAT | 混合 | 返回匹配点对，需额外处理 |
| STalign | 非线性 | 仅支持成对对齐；需单细胞分辨率 |
| CAST | 混合 | 支持跨平台；仅成对对齐 |
| STAIR | 混合 | 对齐 + 整合 + 3D 重建 |
| SPACEL | 混合 | 利用区域信息，稳定性强 |
| Spateo | 混合 | 多切片表现优异；刚性/非刚性可选 |
| SANTO | 混合 | 粗到细对齐与拼接；仅成对对齐 |

### 2.2 数据集规模

| 指标 | 数量 |
|------|------|
| 切片总数 | 240 |
| 真实切片对 | 260 |
| 模拟切片对 | 35 |
| **对齐任务总计** | **295**（每种方法均执行） |

### 2.3 涵盖的技术平台

10x Visium、10x Visium HD、ST、MERFISH、Stereo-seq、BaristaSeq、STARmap、STARmap PLUS、Slide-seq、Slide-seqV2、Open-ST、Xenium、Xenium 5K、CosMx 等 **14 种** 空间转录组技术。

### 2.4 六大评测维度

```
1. 对齐精度（Accuracy）
   ├── 基因型指标：PCC、Cosine Similarity、SSIM、MI
   └── 地标型指标：区域标签匹配准确率、区域重叠比

2. 计算效率（Efficiency）
   ├── 运行时间
   └── 峰值内存占用

3. 鲁棒性（Robustness）
   ├── 切片重叠比例扰动
   └── 初始旋转角度扰动（15° 递增，0–360°）

4. 下游任务影响（Downstream Impact）
   └── 3D 空间聚类（GraphST）的 ARI、NMI、HOM、COM

5. 困难场景（Challenging Cases）
   ├── 跨平台对齐
   ├── 连续多切片（>10 张）
   └── 大规模高分辨率数据

6. 易用性（Usability）
   └── 可用性、代码质量、文档、可复现性等 30+ 子项
```

---

## 三、常规场景评测结果

### 3.1 10x Visium（人 DLPFC 数据集）

- 3 个个体，各 4 张切片，平均约 3,561 spots，33,538 基因
- **综合排名前四**：PASTE2、SPACEL、STAIR、CAST
- PASTE2、STAIR、SPACEL 在其他 Visium 数据集上亦保持领先

### 3.2 MERFISH（小鼠下丘脑视前区）

- 5 张切片，平均 5,663 细胞，155 基因
- **综合表现最优**：Spateo（刚性/非刚性）、PASTE2
- 与 Visium 结果不同，说明**平台类型显著影响方法排名**

### 3.3 测序类 vs 成像类综合排名

| 平台类别 | 数据集 | 切片数 | 综合前三 |
|----------|--------|--------|----------|
| NGS 类 | ST、Visium、Stereo-seq | 27 | **PASTE2、SPACEL、Spateo** |
| 成像类 | MERFISH、BaristaSeq、STARmap | 44 | **Spateo、STAIR、SPACEL** |

**效率权衡**：PASTE2、SPACEL 精度高但耗时较长；Spateo 内存消耗略高。若优先追求精度，可接受效率折中。

### 3.4 指标一致性

Kendall's tau 相关系数显示，多数精度指标排名高度一致（>0.7），评测结果可靠；少数指标间存在互补性（<0.5）。

---

## 四、下游任务影响

以 GraphST 3D 空间聚类为下游任务（DLPFC 数据集）：

- **12/15** 种对齐设置优于未对齐基线
- **SPACEL、CAST、PASTE2** 下游表现最佳
- 各指标（ARI、NMI 等）较基线提升约 **0.3**

**结论**：高质量对齐显著改善 3D 空间域识别；在空间重建工作流中，对齐方法应严格比较和筛选。

---

## 五、鲁棒性评测

### 5.1 切片重叠比例扰动

- 多数方法随重叠比例增加精度提升
- **SPACEL** 在各条件下保持高精度，稳定性突出

### 5.2 初始旋转角度扰动

| 鲁棒性高 | 角度敏感 |
|----------|----------|
| PASTE2、STAIR、SPACEL、Spateo、SANTO、STAligner（最优 landmark） | PASTE、SLAT |

- 初始角度偏差在 **±30°** 内通常对齐效果更好
- **PASTE2 + SPACEL** 在两种扰动下综合最稳

---

## 六、困难场景评测

### 6.1 跨平台对齐

**数据集 1**：小鼠嗅球 Stereo-seq vs Slide-seqV2（2 张切片）

- **STAligner、CAST** 表现最优，SLAT 亦较好

**数据集 2**：10 大平台小鼠脑冠状切片（MERFISH、Xenium、Visium、Stereo-seq、CosMx 等）

- 空间分辨率从亚细胞到 100 μm，基因 panel 差异大
- 多数方法可运行但效果差
- 仅 **MERFISH ↔ Xenium ↔ Xenium 5K** 之间略可接受
- **跨平台对齐仍是当前最大短板**

### 6.2 连续多切片

| 数据集 | 切片数 | 最优方法 |
|--------|--------|----------|
| Stereo-seq Flysta3D | 16 | **Spateo** |
| Open-ST 人转移淋巴结 | 19（~100 万细胞） | **Spateo** |
| MERFISH | 33 / 129 | **Spateo** |

其他方法仅在特定数据集表现尚可，或切片数过多时无法运行。

### 6.3 大规模高分辨率数据

以 Xenium 人乳腺癌 2 张切片为例：

- 仅少数方法能完成对齐，几乎无一满意
- 高分辨率 + 大规模数据易触发 **内存溢出**

**三种缓解策略**：

| 策略 | 说明 | 示例 |
|------|------|------|
| **粗预对齐（PA）** | 手动观察 + 刚性变换，统一初始角度 | STalign 预对齐后近完美对齐且大幅提速 |
| **参数优化（BestPara）** | 系统调参（如 Spateo） | 显著改善对齐效果 |
| **降采样 + 分辨率恢复（DR）** | 降分辨率对齐，再映射回原始分辨率 | 解决内存瓶颈；PA+DR 组合效果更佳 |

---

## 七、易用性评测

基于标准化可用性框架（30+ 子项，含可用性、代码质量、文档、可复现性等）：

| 排名 | 方法 |
|------|------|
| 最易用 | **Spateo、PASTE 系列** |
| 中等 | 多数方法 |
| 门槛较高 | GPSA、SANTO（对编程经验不足用户） |

---

## 八、消融分析：对齐依赖什么信息？

| 场景 | 结果 |
|------|------|
| 仅用空间坐标（打乱基因表达） | SLAT、PASTE_p0 仍可对齐；其他方法失败 |
| 仅用基因表达（打乱空间位置） | **全部方法失败** |
| 仅用基因表达 + 消除密度差异 | 多数方法仍可全局对齐 |

**结论**：现有方法最依赖 **基因表达相似性** 与 **相对空间位置关系**；局部形态特征（如细胞密度）作用有限。

---

## 九、实用选型指南（论文 Fig. 6）

### 9.1 快速选型

```
测序类（NGS）数据        → 优先 PASTE2
成像类数据              → 优先 Spateo
有区域注释标签          → SPACEL
跨平台对齐              → CAST / SLAT
仅需匹配点对、不需新坐标 → SLAT
单细胞分辨率 + 密度结构  → STalign
组织边界清晰            → STAligner
```

### 9.2 推荐工作流

```
1. 数据准备
   ├── 统一为 AnnData 格式
   ├── 质控（过滤细胞/基因）
   └── 粗预对齐（校正初始切片方向）

2. 正式对齐
   ├── 按平台选择首选方法
   ├── 参数调优
   └── 内存溢出 → 降采样 / 进一步过滤

3. 对齐后检查
   ├── 视觉检查
   ├── 指标评估（PCC、SSIM、MI、区域重叠比等）
   └── 不满意 → 换方法重试

4. 下游分析
   └── 3D 聚类、空间域识别、细胞通讯等
```

### 9.3 特殊场景建议

| 场景 | 建议 |
|------|------|
| 内存溢出 | 降分辨率 → 对齐 → 恢复分辨率 |
| 对齐质量差 | 参数调优 → 换方法 → 手动 landmark |
| 跨平台 | 目前无成熟方案，CAST/STAligner 可尝试 |
| 超大数据 | 预对齐 + 降采样 + 参数优化组合使用 |

---

## 十、未来发展方向（论文观点）

1. **跨平台对齐**：生物学意义重大，但现有方法普遍不足，亟需新算法
2. **跨组学对齐**：整合 ST + 影像 + 表观组 + 空间蛋白组，在物理 3D 空间中共配准
3. **时空组学对齐**：跨发育阶段、跨时间点的 ST 数据对齐
4. **大规模数据**：亚细胞分辨率下数据量指数增长，需兼顾速度与精度
5. **大语言模型**：LLM 在单细胞和空间组学中已展现潜力，有望辅助对齐任务

---

## 十一、SABench 工具包

作者将评测框架封装为 **SABench**，支持：

- 快速计算评测指标
- 与现有 11 种方法直接对比
- 用新数据 benchmark 新方法
- 大规模数据的交互式粗对齐、网格降采样、分辨率恢复
- 方法特异性参数调优策略

**GitHub**: [https://github.com/Yunzhi-Yan/SABench](https://github.com/Yunzhi-Yan/SABench)  
**Zenodo**: [https://doi.org/10.5281/zenodo.18605715](https://doi.org/10.5281/zenodo.18605715)

---

## 十二、总结

| 维度 | 核心结论 |
|------|----------|
| 总体 | **没有万能冠军**，方法选择高度依赖平台与场景 |
| NGS 类 | PASTE2、SPACEL 精度领先 |
| 成像类 | Spateo 综合最优 |
| 鲁棒性 | PASTE2、SPACEL 最稳定 |
| 多切片 | Spateo 优势明显 |
| 跨平台 | 当前方法普遍不足，是重要发展方向 |
| 大数据 | 需预对齐 + 降采样 + 参数优化 |
| 下游 | 好对齐显著提升 3D 空间聚类精度 |
| 工具 | SABench 提供可复现的评测框架 |

本研究为空间转录组切片对齐提供了迄今较全面的方法学参考，对 3D 组织重建、跨切片整合和多平台数据融合具有直接指导价值。

---

## 参考文献

```
Yan Y, Gu T, Sun C, Zhang Y, Cui Y, Lin S, et al.
Benchmarking alignment methods for spatial transcriptomics data.
Nature Computational Science. 2026.
doi: 10.1038/s43588-026-00977-z
```

**相关链接**

- 论文: https://doi.org/10.1038/s43588-026-00977-z
- SABench: https://github.com/Yunzhi-Yan/SABench
- Zenodo 数据: https://doi.org/10.5281/zenodo.18605715

**主要对标方法原文**

- PASTE: Zeira et al., Nat Methods, 2022
- PASTE2: Liu et al., Genome Res, 2023
- STAligner: Long et al., Nat Commun, 2022
- Spateo: Qian et al., Cell, 2024
- SPACEL: Li et al., Nat Commun, 2023
- CAST: Fan et al., Nat Methods, 2023
- SANTO: Li et al., Nat Commun, 2024
