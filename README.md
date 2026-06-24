# 空间转录组学习笔记

## 0.Overview of commonly used ST platforms

| Platform (type) | Resolution and panel type | Sample types | RNA quality | Best use cases |
|:---|:---|:---|:---|:---|
| **Visium (FF, sequencing)** | ~55 µm; whole transcriptome | Human, mouse, all species (polyA⁺) | RIN ≥ 7 (≥4 w/ CytAssist) | Broad discovery in fresh tissue |
| **Visium (FFPE, sequencing)** | ~55 µm; whole transcriptome | Human, mouse FFPE | DV200 ≥ 50% (≥30% w/ CytAssist) | Archived samples; full profiling |
| **Visium HD (sequencing)** | ~2 µm; whole transcriptome | Human, mouse FFPE or OCT | RIN ≥ 4; DV200 ≥ 30% | High-res + whole transcriptome |
| **Illumina StrataMap Spatial (sequencing)** ★ | ~1 µm features; cellular resolution; whole transcriptome (polyA⁺, coding + long ncRNA) | Eukaryotic species; **FF (OCT) only** (FFPE in development, ~2027) | FF optimized; vendor has not published a fixed RIN/DV200 cutoff | Unbiased whole-transcriptome at cellular resolution; large capture area (up to 50 × 15 mm); multi-section / cohort-scale profiling within Illumina NGS + DRAGEN/ICM workflow |
| **Xenium (imaging)** | Subcellular; targeted (up to 5000 genes; customizable) | Human, mouse FFPE or fresh-frozen; non-model (custom) | DV200 ≥ 10% | Cell typing; high-res profiling; cross-species (custom panels) |
| **CosMx (imaging)** | Subcellular; targeted panels (up to 6K) or whole-transcriptome (~18K genes, WTX) | Human, mouse FFPE or fresh-frozen | DV200 ≥ 10% | Multiplexed profiling; spatial cell state mapping |
| **MERFISH / seqFISH (imaging)** | Subcellular; highly multiplexed (customizable) | Fresh-frozen; FFPE (with optimization) | Protocol-dependent | Deep profiling in microscopy-capable labs |
| **Stereo-seq (sequencing)** | 500 nm; whole transcriptome (species-specific probes) | Human, mouse, non-model (custom probes) | RIN ≥ 7 recommended | Nanoscale mapping; large area profiling |
| **Non-model species** | Varies by platform and probe design | Visium (polyA⁺), Xenium, Stereo-seq, **StrataMap (polyA⁺, any eukaryote)** | Variable | Cross-species studies (requires custom panels or transcriptomes) |

[Grases D, Porta-Pardo E. A practical guide to spatial transcriptomics: lessons from over 1000 samples[J]. Trends in biotechnology, 2025.](https://www.cell.com/trends/biotechnology/abstract/S0167-7799(25)00357-9)

<img src="./0.platform/Overview_of_spatial_transcriptomics.png" height=700 width=500>

[Heumos L, Schaar A C, Lance C, et al. Best practices for single-cell analysis across modalities[J]. Nature Reviews Genetics, 2023, 24(8): 550-572.](https://www.nature.com/articles/s41576-023-00586-w)

<img src="./0.platform/spatial-transcriptomics.png" height=500 width=600>

[Williams C G, Lee H J, Asatsuma T, et al. An introduction to spatial transcriptomics for biomedical research[J]. Genome medicine, 2022, 14(1): 68.](https://link.springer.com/article/10.1186/s13073-022-01075-1)

![technologies_of_spatial_transcriptomics.png](./0.platform/technologies_of_spatial_transcriptomics.png)

[Rao A, Barkley D, França G S, et al. Exploring tissue architecture using spatial transcriptomics[J]. Nature, 2021, 596(7871): 211-220.](https://www.nature.com/articles/s41586-021-03634-9)

<img src="./0.platform/Workflow.jpg" height=1000 width=600>

[Yue L, Liu F, Hu J, et al. A guidebook of spatial transcriptomic technologies, data resources and analysis approaches[J]. Computational and Structural Biotechnology Journal, 2023, 21: 940-955.](https://www.sciencedirect.com/science/article/pii/S2001037023000156)

---