# 05 — Cell typing

## Part A: cell-level labels (05a_cell_typing.ipynb)

### What we did
- Reference: MSK SPECTRUM (CELLxGENE h5ad, 927,205 cells, 41 treatment-naive HGSOC patients, 10x 3' v3, CD45+/- sorted). Balanced subset: <=12,000 cells per author_cell_type, adnexa preferred, doublets removed -> 90,071 cells, raw counts, gene symbols.
- CellTypist (logistic regression, SGD, feature selection) trained on reference; predicted on 190,402 bin2cell cells (17,388 common genes).
- scANVI: first attempt with reference+query in one model and 'tech' as batch collapsed (query spread across rare classes: over-integration). Second attempt: reference-only SCVI/SCANVI (donor as batch, self-accuracy 0.974), then query mapping (scArches, load_query_data) -> realistic labels.

### Key numbers
- scANVI (query-mapped): tumor 86.7%, fibroblast 9.9%, endothelial 2.3%, monocyte 1,306, plasma 567, T 130, mast 120, B 28. Agreement with CellTypist 95.7%; disagreements sit on tumor-stroma boundaries.
- Lymphocytes are genuinely rare: PTPRC>=3 in 21 cells, CD3E>=2 in 111 (immune-excluded tumor).
- Macrophages exist (CD68>=3: 2,167 cells, 1.1%) but are mislabelled as tumor: 82% under CellTypist, 66% under scANVI. Their UMI (4,328) and area (116 µm²) are above average: mixed profiles from spillover.
- Marker means per label are diagonal (each label maximal in its own markers), but WFDC2 ~1.8 in all stromal labels (tumor 2.8): pervasive tumor spillover at gene level.
- Confidence scores saturate at 1.0 for both tools; used method disagreement as the uncertainty flag instead.

### Lessons
- Batch correction assumes shared composition; reference (balanced) vs HD (89% tumor) violates it -> over-integration. Train the reference alone and map the query (scArches).
- Majority voting erases rare scattered cell types in spatial data; use per-cell predictions.
- Reported probabilities from SGD logistic regression and scANVI are not calibrated here; check the distribution before using them.
- Cell typing in HD is limited by spillover, not by the classifier: abundant compartments are correct, sparse cells surrounded by tumor (macrophages) are absorbed. Before/after correction, 'fraction of CD68+ cells labelled monocyte' is a direct validation metric for the correction tool.
- Environment: RAPIDS (cuml/cudf) preinstalled on Colab GPU runtimes breaks CellTypist import under pandas 3 -> block with sys.modules; install scvi-colab at session start, never mid-session (torch circular import); obs_names read back from CSV must be cast to str before joining.

### Decision
- Primary labels: ct_scanvi (query-mapped); secondary: ct_celltypist; flags: label_agree, macrophage_candidate. Saved in processed/cells_bin2cell_typed.h5ad.

## Part B: bin-level deconvolution (05b_bin_deconvolution.ipynb)

### What we did
- cell2location RegressionModel on the 90k reference (donor as batch, 14,871 genes after filter): 250 epochs, 37.7 min on A100 (data-loading bound). Signatures correct (WFDC2 tumor 46.8 vs fibroblast 0.9; COL1A1 fibroblast 164; CD68 monocyte 8.3).
- Cell2location on a random 60,000-bin subsample of 8 µm bins (13,203 genes), N_cells_per_location=1, detection_alpha=20, 2,500 epochs (16 min); ELBO still decreasing at stop (under-trained; recommended 10-30k epochs).

### Key numbers
- Dominant type per bin: tumor 83%, fibroblast 10.7%, endothelial 4.9%, monocyte 1.1%. Total abundance per 8 µm bin: median 0.64 cells.
- Agreement between bin dominant type and nearest scANVI cell label (cell within 8 µm, 71.5% of bins): 93.8% overall; tumor 98%, endothelial 79%, fibroblast 72%, monocyte 61%.
- Mixed bins (2nd type > 30% of 1st): 49% overall; fibroblast-dominant 82%, endothelial 89%, monocyte 89%, tumor 42%.

### Lessons
- Two independent routes (segmentation + classifier vs deconvolution) agree on major compartments; disagreement concentrates in sparse types and boundaries.
- Half of all 8 µm bins are mixtures; stromal bins almost always carry tumor signal. Deconvolution quantifies mixing but cannot tell physical spillover from true co-occupancy.
- Use abundance vectors, not dominant labels, for any bin-level downstream analysis.
- Train cell2location to convergence (watch the ELBO curve), not to a time budget; full-slide run (~2 h on A100) reserved for project data.
- Compare categorical labels from different sources as strings (pandas refuses to compare categoricals with different category sets).
