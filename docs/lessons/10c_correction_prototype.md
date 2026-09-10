# 10c — Spillover correction prototype (Richardson-Lucy / EM) on the synthetic slide

Correction uses only blind estimates from 10b (lambda 2.5 µm, ambient 0.383 UMI/bin); the true lambda (4.0) is never used.

## Method
y = K x + a + Poisson noise. K from cell positions and the estimated exponential PSF; a from the far-field plateau.
Multiplicative EM update x <- x (K^T (y / (Kx + a))) / (K^T 1). The K^T 1 term is the per-cell coverage, so cells at the organoid edge (low coverage) are scaled up automatically.

## What worked: the scale problem
- Total UMI per cell, correlation with truth (log): **0.665 -> 0.898** (about 70% of the error removed).
- Median total UMI: true 3,234, measured 2,800 (-13%), corrected 3,407 (+5%).
- **The artificial edge-core gradient is removed**: UMI ratio to truth by distance from the edge goes from 0.563 / 0.672 / 0.759 / 0.717 / 0.792 to 1.053 / 1.012 / 1.033 / 1.038 / 1.017. Every band within 5% of 1.0.
- Total signal recovered: measured 15.2M UMI, corrected 20.2M, true 19.8M (error 23% -> 2%).
- Correction is tolerant to PSF error: lambda underestimated by 38% (2.5 vs 4.0) produced only ~5% over-correction.
- Early stopping is real: correlation peaks at iteration ~6 (0.905) and declines slowly (0.898 at 20). Real data will need a stopping rule (cross-validation or a regularisation term) since no ground truth is available.

## What did not work: the composition problem
- Per-cell profile correlation with truth: **0.815 -> 0.781 (worse)**. Clustering ARI 0.319 -> 0.333 (unchanged). Gene detection at the edge 0.802 -> 0.805 (unchanged); core inflation 1.033 -> 1.002 (improved).
- Reasons, in order: (1) the gene-level EM was seeded with each cell's *measured* profile to keep memory feasible, so the mixing was baked into the forward model - a setup error, not an algorithm failure; (2) per-gene counts are 0-3 UMI, where multiplicative EM amplifies noise; (3) genes lost at the edge are genuinely below detection and cannot be recovered.

## The key conceptual split
Spillover correction contains **two problems of different difficulty**:
- **Scale**: how much signal a cell lost. Solvable with the current model; the edge artefact disappears.
- **Composition**: which transcript belongs to which cell. Not solved by cell-level EM; needs bins x genes observations directly, or gene-module-level solving, or a prior on neighbour profile similarity.

## Decision for the thesis
- Tool v1 scope: per-cell scale correction + edge compensation + ambient removal + parameter QC (PSF, ambient, striping per slide). This is demonstrably feasible and fixes a documented artefact.
- Composition correction is deferred to v2 / a separate research question, with the failure mode documented.
- Never evaluate a correction on a single metric: totals improved while profiles degraded. Report a metric panel.

## Compute notes
- Gene-level EM (17,595 genes, 8 iterations, blocks of 500) took 51 min on CPU with dense intermediates; a real implementation needs sparse intermediates or GPU.
- Reading the h5ad with anndata fails under pandas 3 (StringDtype na_value); counts and gene names were read directly with h5py from layers/counts and var/_index/values.
