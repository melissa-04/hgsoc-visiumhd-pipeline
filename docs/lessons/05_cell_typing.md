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

## Part B: bin-level deconvolution (05b) — pending
