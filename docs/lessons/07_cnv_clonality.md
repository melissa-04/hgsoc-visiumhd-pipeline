# 07 — CNV inference and clonality (infercnvpy on bin2cell cells)

## What we did
- Gene positions parsed directly from GENCODE v46 GTF (17,518/17,595 genes placed); no gtfparse.
- infercnvpy (window 200 genes, step 10), reference = fibroblast + endothelial + monocyte (24,388 cells); run in its own session with n_jobs=1, outputs saved to disk; clustering done in a fresh session (scikit-learn only).
- Clone search: PCA(20) on the CNV matrix of 165,169 tumour cells, k-means k=2..8 with silhouette; k=2 split characterised by profile correlation and amplitude.

## Key numbers
- CNV score: tumour 0.0064 vs reference 0.0041-0.0045 (contrast only ~1.4x: reference contaminated by tumour spillover, tumour diluted by stroma).
- Chromosome-level gains: chr19 (0.019), chr12p, chr20, chr6, chr1, chr7; losses: chr13, chr5, chr21. Arm-level chr8: 8p loss / 8q gain. chr17: focal peak at window 35/82 (0.028, strongest signal genome-wide) at the ERBB2/GRB7 (17q12) region, present in all tumour cells -> clonal focal amplification (matches step 01 observation; needs WES/FISH confirmation).
- Silhouette max 0.16 at k=2 and decreasing: no discrete subclones at this resolution. k=2 groups: profile correlation 0.956, amplitude ratio 0.61 -> same clone, different signal amplitude.
- Tumour CNV score vs distance to stroma: 0.0050 at 0-8 µm -> 0.0066 plateau beyond 32 µm (25% dilution at boundaries, same ~30 µm spillover scale).
- CellCharter domains C5/C0 share the core CNV profile -> their expression program is not clone-driven (technical/regional-state suspicion remains).

## Lessons
- Read chromosome arms, not chromosome means (8p/8q and 17p/17q cancel).
- A k-means split of CNV profiles is a clone only if groups differ by chromosome sign; identical shape with lower amplitude is dilution/quality.
- Spillover attacks CNV from both sides (dirty reference, diluted tumour); boundary cells lose ~25% of CNV signal.
- Allele-based CNV (Numbat/CalicoST) impossible with probe-based chemistry; relevant for the HD 3' decision.
- Environment: infercnvpy's fork-based process pool deadlocks later numba/BLAS calls in the same session (scanpy neighbors, sklearn PCA). Run it alone, save, restart. pandas 3 Arrow strings must be disabled (future.infer_string=False). Colab 'Interrupt' can reset the kernel: write intermediates to disk before long steps.

## Decision
- Single dominant clone with 17q12 focal amplification; carry cnv_score per cell; no clone labels used downstream. Clone-level analyses wait for project WES.
