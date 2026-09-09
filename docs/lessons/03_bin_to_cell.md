# 03 — Bin to cell (2 µm -> cells)

## What we did
1. Loaded Space Ranger 4.0.1 segmentation (183,450 cells; nucleus + cell polygons) and checked polygon/image alignment (hematoxylin-under-mask shift test: peak at 0,0).
2. Marker purity and compartment contamination for 8 µm bins vs SR cells (small/large thirds).
3. Contamination vs distance to the other compartment.
4. bin2cell 0.3.4 on 7,007,169 2 µm bins: destripe (fixed for pandas 3), StarDist 2D_versatile_he on 0.5 mpp H&E (194,386 nuclei), expand 2 bins, bin_to_cell (191,343 cells).
5. Striping at 2 µm before/after destripe; hematoxylin-per-label false-positive check; bin-grid vs image registration with MT genes.

## Key numbers
- SR cells: median 3,277 UMI, 2,129 genes; cell area 112 µm², nucleus 28 µm²; 87% of tissue UMI assigned.
- bin2cell: median 3,274 UMI (destriped scale), area 84 µm², nucleus 38.5 µm²; 72% of UMI assigned.
- Purity did not improve from bins to cells (0.826 vs 0.829). Fibroblast signal in epithelial-dominant units: bins 16.4%, SR 16.0%, small SR cells 15.2%, large 16.8%, bin2cell 15.3%.
- Contamination decays with distance to the other compartment, symmetric in both directions: ~32% at 0-8 µm -> ~13% at 32-48 µm -> ~8% floor beyond 100 µm; e-folding ~20-25 µm (same scale as the off-tissue edge decay).
- Tighter expansion (bin2cell) lowers boundary contamination by 5-6 points in the first 1-2 cells only; curve shape and floor unchanged.
- Striping at 2 µm: row sd 0.247 (21.7x null, lag1 -0.49); after destripe 0.032 (4.9x null, lag1 -0.04). Typical residual line effect ~3%.
- Bin grid vs image registration: MT-gene minimum at (0,0); nucleus/cytoplasm contrast only 14% at 2 µm.
- Probe set lacks MALAT1/NEAT1/XIST.

## Lessons
- Two segmentations, two error profiles: SR under-sizes nuclei and misses pale stromal nuclei; StarDist at prob 0.01 catches full nuclei but labels some vacuoles. Hematoxylin per label is unimodal, so stain alone cannot separate false positives; flag, do not drop.
- Moving from bins to cells buys countability, not cleanliness: cross-compartment mixing is ~16% and mostly segmentation-independent.
- Mixing is distance-dependent and symmetric -> physical spillover, not boundary biology. Best segmentation still leaves ~25% foreign signal in boundary cells. This is the motivation for a distance-aware correction model.
- Quantile-based positivity is not comparable across subsets with different composition; use fraction-based contamination measures.
- Do not compare raw counts between destriped and non-destriped matrices; compare ratios.
- Destripe removes the alternating line pattern but leaves ~3% line effects; a nucleus-anchored or jointly estimated row x column term is justified.
- Registration checks: polygon-vs-image is trivially satisfied for SR; the meaningful test is bin-grid-vs-image using counts (nuclear-retained or cytoplasm-restricted genes).
- Subcellular claims at 2 µm are not supported in this dataset (14% nucleus/cytoplasm contrast).
- Environment: TF 2.20 + Keras 3 breaks StarDist 0.9 (needs tf-keras + TF_USE_LEGACY_KERAS=1, run in a subprocess); numpy 2 breaks csbdeep normalize on uint8 (convert to float32); bin2cell destripe_counts breaks on pandas 3 (scale manually); never name a working folder after a package.

## Decision
- Carry both cell matrices forward; default to bin2cell (expand 2) for cell-level steps, SR as comparison.
- ENACT and FICTURE not run (logged).
