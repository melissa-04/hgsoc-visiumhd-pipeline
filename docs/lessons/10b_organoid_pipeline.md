# 10b — Running the pipeline blind on the synthetic organoid slide

Analyses use only the measured data; ground truth is used solely for evaluation.

## Organoid detection
- Otsu on log UMI (8 µm bins) finds exactly 24 objects; area recall 0.83, precision 0.91, but **99.5% of cells are captured** (edge cells 99.3%). Area-based accuracy is misleading for organoids; count cells instead.
- Threshold choice barely matters (83 vs 60 UMI give the same result) because inside/outside contrast is ~200x.
- Diameter from the count mask is biased: **+20 µm for 100-150 µm organoids** (spillover halo is pulled inside), and irregular organoids are split into fragments (ORG08 -92 µm). Median error +7.9 µm. Measure size from H&E, not from counts.

## Edge gradient as the analyst sees it
- Cells 0-5 µm from the edge: measured/true UMI **0.597**; beyond 10 µm 0.85-0.94.
- Genes detected behave differently: 0.84 at the edge but **1.15 in the core** - spillover adds genes the cell does not express. 'Genes per cell' is not a clean quality metric under spillover.
- Profile correlation with truth is 0.80-0.84 everywhere, slightly higher at the edge (fewer neighbours, purer profile). Normalisation removes the depth difference but not the mixing.
- Summary: edge cells are **few but pure**, core cells **many but mixed**. Count-based analyses penalise the edge, profile-based analyses penalise the core.

## Bin size does not help
- edge/core UMI ratio: 0.558 (2 µm), 0.413 (8 µm), 0.479 (16 µm). Larger bins straddle the boundary and mix inside with outside; at 16 µm a 100 µm organoid is only ~30 bins. Consistent with the Stereo-seq organoid benchmark (iScience 2026). Use 2 µm or cells for edge-core work; 8/16 µm only for whole-organoid summaries.

## Clustering
- k-means (k=10) on measured vs true counts: the most edge-enriched cluster has 23% edge cells vs 15.9% overall - no meaningful separation. **The gradient does not create an edge cluster** (prediction refuted).
- But ARI(measured, true) = 0.422: the same cells, same pipeline, only measurement noise added, and half the cluster assignments change. Unsupervised cluster identity in HD is fragile.
- The most edge-enriched measured cluster is marked by AKIRIN1, MACF1, TPGS2, MYCBP, NDUFS5, RRAGC, PPIE - **the same signature as the CellCharter C5/C0 domains in the real data (step 06)**. Since the synthetic data contain no biological difference, that real-data signature is a technical artefact. Simulation explained a real-data finding.

## Blind parameter estimation (feasibility for the correction tool)
- Ambient: estimated 0.383 vs true 0.364 UMI per 2 µm bin (ratio 1.05). The outside profile plateaus beyond ~48 µm and stays flat to 200 µm; 89% empty area makes this nearly exact.
- PSF: forward-model fit gives 2.5 µm (true 4.0); loss curve is flat between 1 and 5 µm and rises sharply beyond, so report an interval, not a point. Gaussian kernel fitted to an exponential truth explains the downward bias.
- **Naive reading of the edge-decay constant gives 9.2 µm - 2.3x the true value.** The observed decay is a superposition of many cells' PSFs, not the per-cell PSF. Applying the same factor to the ~18 µm decay measured in real tissue (step 01) gives ~8 µm, consistent with the 3-5 µm from calibration.
- Striping: corr(estimated, true) 0.87 rows, 0.85 columns; extremes shrunk toward the mean (safe direction).
- A bump in the outside profile at 14-20 µm is a diagnostic that the detected mask is too tight.

## Per-organoid statistics (24 samples)
- Null test (random split): 0 significant genes in both measured and true counts. With 24 independent samples the pseudobulk workflow is calibrated - the 0.5-2% false positives seen in step 09 came from having only 8 spatially-blocked pseudo-samples.
- Size effect (small <=150 µm vs large >=240 µm): **0 significant genes** in both. The 9 µm/diameter signal loss is proportional across genes, so size factors absorb it (prediction refuted).
- Measurement does remove ~1,200 genes from testable range (12,330 vs 13,569 genes past the count filter): low-expressed genes are diluted below detection.

## Conclusions for the project
- Spillover damages cell-level analyses, not sample-level pseudobulk means. Do organoid comparisons at pseudobulk level with organoid as the sample; report cells per organoid.
- Do not compare edge vs core without correction: a 1.6-fold artificial count gradient sits underneath.
- Do not derive organoid size from counts; do not use 8/16 µm bins for regional work.
- Treat AKIRIN1/NDUFS5/MACF1-type cluster signatures as technical until proven otherwise.
