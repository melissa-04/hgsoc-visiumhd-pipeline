# 01 — Setup and loading (Visium HD, 8 µm bins)

**Dataset:** 10x Visium HD, human ovarian cancer (papillary serous carcinoma), fresh frozen, Space Ranger 4.0.1, deep sequencing (2.3B read pairs).

## What we did
1. Environment on Colab Pro (High-RAM: 50 GB RAM, 24 CPU, 220 GB local disk). Raw data on Drive; extracted to local disk per session.
2. Loaded 8 µm filtered matrix into AnnData, joined bin coordinates and scale factors.
3. Bin-level QC: total UMI, genes per bin, % mitochondrial; histograms and spatial maps on H&E.
4. Raw matrix: off-tissue background, edge-decay curve, ambient gene profile.

## Key numbers
- 445,751 in-tissue bins x 18,132 genes (probe-set genes with included probes); sparsity 0.93.
- Median UMI per 8 µm bin: 1,773; median genes: 1,296; median %MT: 7.0 (12 MT genes in probe set).
- Raw grid: 702,244 bins; 256,493 off-tissue. Off-tissue median 13 UMI, 90th percentile 200.
- Edge decay (median UMI vs distance from tissue): 390 (0-8 µm) -> 240 -> 146 -> 93 -> 54 (32-48 µm) -> 18 (64-96) -> 7 (300-500) -> 3 (>1 mm).
  e-folding length of the fast component ~18 µm; no flat plateau, long tail persists to 1 mm.
- Relative to in-tissue median (1,802): bins 1/2/3/4 from the edge carry 22% / 13% / 8% / 5%.
- Ambient composition ~ tissue composition (ratios 0.9-1.1); MT genes slightly enriched.

## Lessons
- Counts are UMIs, not reads. Bin != cell. Sparsity grows as bin size shrinks.
- Two coordinate systems (array index vs image pixels); always use pxl_* columns and the right scale factor for the image you draw on.
- Filtered matrix = Space Ranger's tissue call. Check it: draw the capture-area boundary first (tissue outside the 6.5 mm square is simply unmeasured, not 'excluded').
- HD QC is spatial: a low-count bin is biology in necrosis and an artifact inside a tumor nest. Do not use single-cell thresholds blindly; %MT alone is not a filter for bins.
- Off-tissue signal is real and distance-dependent: spillover concentrates within ~30-50 µm of boundaries; far-field ambient is ~0.4% of tissue median. Correction must be distance-aware, not a flat background subtraction.
- Genes-per-bin histogram had a shoulder -> two compartments (tumor nests vs stroma), confirmed spatially.
- Biological side note: ERBB2 and GRB7 (17q12) among top tissue genes -> possible HER2 amplification; test in CNV step.

## Open questions for step 02
- Recompute the edge curve with an H&E-derived tissue mask (Space Ranger mask may be 1 bin too tight).
- Separate 'distance to tissue' from 'distance to capture-area edge' in the far tail.
- Inspect small off-tissue fragments called in_tissue=1 (bottom-right).
