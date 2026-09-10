# 09 — Multi-sample analysis and paired statistics (deep vs minimum depth, same section)

## What we did
- Downloaded the minimum-depth version of the same section (441.5M vs 2,316M read pairs; same slide, same segmentation).
- Compared metrics_summary; matched 183,443 cells present in both; applied the same stored CellTypist model to both.
- Pseudobulk per cell type; correlation between depths.
- Built 8 spatial pseudo-samples; pseudobulk per (depth x block x cell type); PyDESeq2 with a null test (random split of deep samples) and a paired design (~block + depth).

## Key numbers
- Depth 0.191 of reads -> UMIs 0.335, genes per bin 0.394, median UMI per cell 3,277 -> 1,097, median genes 2,129 -> 864. Sequencing saturation 0.564 vs 0.235 explains the non-linear loss. Total genes detected barely moves (0.981).
- Cell count, cell/nucleus area, tissue fractions identical: segmentation is depth-independent.
- Per-cell label agreement 95.4%, but composition shifts: fibroblast 9.75% -> 7.70%, tumour 87.2% -> 88.8%, B cells 293 -> 1,353 (a low-information class acting as a sink), T cells 4 -> 0. Fibroblast->tumour reassignment 3,611 cells.
- Pseudobulk logCPM correlation between depths: tumour 0.9997, fibroblast 0.9991, endothelial 0.9967, but monocyte 0.82 (154 cells), plasma 0.68 (239), B 0.46 (293).
- Null test (no real difference): 78/15,422 significant genes in tumour, 286/13,520 in fibroblast at padj<0.05.
- Paired depth test (~block + depth): 5,057/13,161 genes in tumour, 325/7,886 in fibroblast. Dispersion trend fit did not converge (8 samples).

## Lessons
- Depth affects per-cell analysis strongly and pseudobulk analysis weakly - provided each group has thousands of cells. Below ~1,000 cells pseudobulk is also unreliable.
- Cell-type composition is depth-sensitive and shifts systematically (weak signal drifts to the dominant class or to low-information classes). Never compare composition across samples sequenced at different depths without matching depth or modelling it.
- Always run a null test: random grouping of our pseudo-samples produced 0.5-2% significant genes, driven by regional biology and unstable dispersion estimates with n=8.
- Significance is not effect size: with 160k cells per group, tiny systematic shifts reach padj<0.05. Filter on padj and log2FC.
- Paired design (~block + condition) is the prototype for the project (~patient + condition); it absorbs sample-level baselines.
- Environment: pandas 3 breaks anndata's read_10x_h5 (StringDtype na_value) -> read 10x .h5 directly with h5py; mid-session pip installs corrupted pandas.reshape -> compute crosstabs with numpy; run PyDESeq2 in a subprocess.

## Decision
- Project pipeline: statistics at pseudobulk level with ~patient + condition; report cells per group; match sequencing depth across samples or include it as a covariate; run a null test before every real comparison.
