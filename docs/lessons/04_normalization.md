# 04 — Normalization (8 µm bins and bin2cell cells)

## What we did
- Units with <50 UMI (bins) / <100 UMI (cells) removed before model-based normalization (0/0 residuals otherwise).
- 2,000 HVGs (seurat_v3 on raw counts) shared by all methods.
- Shifted log (median-depth scaling + log1p), analytic Pearson residuals (theta=100), scVI (nb likelihood, 30 latent, 60 epochs, 2.4 min on A100, cells only).
- Metrics: |corr(PC, log-UMI)|, Leiden agreement at matched cluster number (ARI), depth R2 explained by clusters, Moran's I of markers, cluster maps.

## Key numbers
- |corr(PC1, logUMI)|: bins log 0.61 -> Pearson 0.09; cells log 0.40 -> Pearson 0.11.
- Depth R2 by ~10 clusters (cells): log 0.28, Pearson 0.12, scVI 0.16.
- Matched-resolution ARI between representations: 0.32-0.43 (cluster identity depends on representation).
- Moran's I of markers preserved (WFDC2 0.45/0.45, DCN 0.27/0.28); COL1A1 slightly lower under Pearson (0.70 -> 0.62) due to residual clipping. PTPRC ~0 (sparse immune cells).
- Same Leiden resolution gives 7-9 clusters (log) vs 18-20 (Pearson): compare at matched cluster numbers only.

## Lessons
- Log normalization leaves depth in PC1/PC2 for HD bins; Pearson residuals and scVI remove it from the leading axis.
- Metrics that depend on cluster count (ARI at fixed resolution, max/min cluster depth) mislead; use matched resolution and scale-free measures (R2).
- Part of the depth-cluster relation is biology (tumor cells carry ~8x more RNA than stroma); expect reduction, not zero.
- Cluster identity is representation-dependent here -> cell typing must be reference-based (step 05), clusters are exploratory.
- scVI shows large spatially contiguous domains (right/left/top). Biology or gradient? Test against CNV clones in step 07.
- bin2cell counts are destriped fractional values; rounded to integers for count models (documented approximation).
- Probe-set gaps: EPCAM and PTPRC not in the HVG set; Pearson residual matrices only cover HVGs, keep log layer for marker plots.

## Decision
- Cells: Pearson-PCA primary; scVI latent stored for integration (step 09) and domain check (step 07); log layer for visualization.
- Bins: Pearson-PCA primary.
