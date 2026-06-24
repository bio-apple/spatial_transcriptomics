# 名词解释



苏木精（Hematoxylin）

1.  H&E（苏木精-伊红）染色：苏木精将细胞核染成深蓝色/紫色，伊红将细胞质染成粉红色，从而让原本透明的组织切片在显微镜下显现出清晰的解剖学和病理学结构。
2.  形态学成像（Morphological Imaging） 的核心目的是在组织被破坏裂解之前，用光学的手段把细胞的结构、边界以及病理特征原汁原味地记录下来。
3.  Open Microscopy Environment TIFF：行业绝对标准：OME-TIFF (*.ome.tif / *.ome.tiff)


Spatially Variable Genes(空间可变基因


# 1.各空间转录组平台官方文件格式对比

> 数据来源说明见文末。带 ✅ 标记的为官方文档已明确记载;带 ⚠️ 标记的为信息来源不完整,需以实际拿到的文件为准。

| 平台(Vendor) | 技术 / 产品线 | 二级分析流程 | 官方文件格式 | 关键附加文件 / 备注 |
|---|---|---|---|---|
| **Illumina** ✅ | Illumina Spatial Technology | DRAGEN Spatial Transcriptome | 三选一(至少满足一种):<br>`pipeline-manifest.json`<br>或 `tar.gz`<br>或 `.h5ad` | 固定附加 `.ome.tiff`;<br>可选 `contour.csv`;<br>每组文件对应一个样本 |
| **10x Genomics** ✅ | Visium(HD WT Panel / HD 3′ / Spatial Gene Expression) | Space Ranger | `.h5`<br>或三件套:`barcodes.tsv.gz` + `features.tsv.gz` + `matrix.mtx.gz` | 需配 `_spatial.tar.gz`(含图像与位置信息);<br>可选高分辨率 `.tif`;<br>每次仅支持导入1个样本 |
| **10x Genomics** ✅ | Xenium(In Situ Gene Expression) | Xenium Analysis | `cell_feature_matrix.h5`<br>`cells.csv.gz`<br>`cell_boundaries.csv.gz`<br>`nucleus_boundaries.csv.gz`<br>`transcripts.csv.gz` | `morphology_focus.ome.tif`(细胞形态图像);<br>需为解压后的 Xenium Output Bundle;<br>每次仅支持导入1个样本 |
| **BGI(华大)** ⚠️ | Stereo-seq | SAW(Stereo-seq Analysis Workflow) | **GEF**(Gene Expression File,二进制,推荐格式,分 `.raw.gef` / `.tissue.gef` / `.cellbin.gef` 等子类型)<br>或 **GEM**(纯文本矩阵,含 `geneID, x, y, MIDCount, ExonCount` 列) | 配套图像文件 `.ipr`(ImageStudio图像记录)、`.tar.gz`(压缩图像);<br>下游可转 `.h5ad`(Stereopy / Scanpy 通用格式);<br>可视化工具为 StereoMap |

**Illumina Spatial Solution:** https://help.connected.illumina.com/icm/reference/supported-data-types

---)