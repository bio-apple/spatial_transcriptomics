# 各空间转录组平台官方文件格式对比

> 数据来源说明见文末。带 ✅ 标记的为官方文档已明确记载;带 ⚠️ 标记的为信息来源不完整,需以实际拿到的文件为准。

| 平台(Vendor) | 技术 / 产品线 | 二级分析流程 | 官方文件格式 | 关键附加文件 / 备注 |
|---|---|---|---|---|
| **Illumina** ✅ | Illumina Spatial Technology | DRAGEN Spatial Transcriptome | 三选一(至少满足一种):<br>`pipeline-manifest.json`<br>或 `tar.gz`<br>或 `.h5ad` | 固定附加 `.ome.tiff`;<br>可选 `contour.csv`;<br>每组文件对应一个样本 |
| **10x Genomics** ✅ | Visium(HD WT Panel / HD 3′ / Spatial Gene Expression) | Space Ranger | `.h5`<br>或三件套:`barcodes.tsv.gz` + `features.tsv.gz` + `matrix.mtx.gz` | 需配 `_spatial.tar.gz`(含图像与位置信息);<br>可选高分辨率 `.tif`;<br>每次仅支持导入1个样本 |
| **10x Genomics** ✅ | Xenium(In Situ Gene Expression) | Xenium Analysis | `cell_feature_matrix.h5`<br>`cells.csv.gz`<br>`cell_boundaries.csv.gz`<br>`nucleus_boundaries.csv.gz`<br>`transcripts.csv.gz` | `morphology_focus.ome.tif`(细胞形态图像);<br>需为解压后的 Xenium Output Bundle;<br>每次仅支持导入1个样本 |
| **BGI(华大)** ⚠️ | Stereo-seq | SAW(Stereo-seq Analysis Workflow) | **GEF**(Gene Expression File,二进制,推荐格式,分 `.raw.gef` / `.tissue.gef` / `.cellbin.gef` 等子类型)<br>或 **GEM**(纯文本矩阵,含 `geneID, x, y, MIDCount, ExonCount` 列) | 配套图像文件 `.ipr`(ImageStudio图像记录)、`.tar.gz`(压缩图像);<br>下游可转 `.h5ad`(Stereopy / Scanpy 通用格式);<br>可视化工具为 StereoMap |

---

## 平台细节说明

### Illumina(StrataMap / Illumina Spatial Technology)
- 这是 **Illumina Connected Multiomics(ICM)** 平台接受的"上传"格式,而不是 DRAGEN 流程一次性输出的全部文件——三种格式(`pipeline-manifest.json` / `tar.gz` / `.h5ad`)是互斥的三选一上传方式。
- `contour.csv` 为可选文件,官方文档未展开说明其具体字段结构。

### 10x Genomics(Visium / Xenium)
- Visium 数据为 spot 级(非单细胞分辨率),一个 spot 可能覆盖多个细胞。
- Xenium 数据为真正单细胞 / 亚细胞分辨率,故同时提供 `cell_boundaries` 和 `nucleus_boundaries` 两层分割边界。

### BGI(华大 / Stereo-seq)
- 技术原理与 10x 不同:采用 DNA 纳米球芯片(DNB)捕获,理论分辨率可达纳米级,实际分析时常按需聚合为不同 bin size(如 bin50、bin100)或做精细的 cell bin(单细胞级)分割。
- **GEF 格式**是 SAW 推荐的主力格式,内部按层级组织(Group/Dataset 结构),按数据粒度分为:
  - **bin GEF**:按方块 bin(如 50×50、100×100 等)聚合的表达矩阵
  - **cellbin GEF**:经过细胞分割后的单细胞级表达矩阵
- **GEM 格式**是纯文本的中间产物,可读性更好但读取速度慢于 GEF,常用于跨工具兼容(比如导入 Stereopy 或自定义脚本时)。
- 官方分析配套工具是 **Stereopy**(Python,类似 scanpy 的 API 风格)和桌面端可视化工具 **StereoMap**。
- 目前 Stereo-seq 数据**未出现在 Illumina ICM 的官方支持列表**中,如需导入 ICM,大概率需要先用 Stereopy 转换为 `.h5ad` 格式(ICM 文档中 `.h5ad` 是标注为"Various 厂商通用"的格式,经 Scanpy 生态产出即可)。
