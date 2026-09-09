# Gap log — tools and gaps found while running the pipeline

| Step | Tool / task | Worked at HD scale? | Time / memory | Gap or note |
|---|---|---|---|---|
| 01 | scanpy read_10x_h5 (8 µm, 446k bins) | yes | <1 min, ~5 GB | fine |
| 01 | tar extraction of binned_outputs | yes | 6 min for 23 GB | hard links to 002um images break when extracting 008um only |
| 01 | off-tissue background / edge decay | custom code | seconds | no existing tool computes this for HD; spillover ~30-50 µm from edges |
| 02 | H&E tissue mask (skimage Otsu / multi-Otsu) | yes | seconds | stain-based masks under-call pale tissue; no tool combines image + counts for HD tissue calling |
| 02 | SpotSweeper-style local outliers | custom (Python) | seconds, 446k bins | R package not run yet; Python re-implementation trivial |
| 02 | striping test (permutation + autocorrelation) | custom | seconds | no existing tool quantifies stripe strength; bin2cell destripe is a correction, not a diagnostic |
| 02 | off-tissue background vs capture-edge distance | custom | seconds | edge-efficiency effect documented; no tool models it |
| 03 | Space Ranger 4 segmentation | provided | n/a | closed model; nuclei under-sized (28 µm²), pale stromal nuclei missed; 87% UMI assigned |
| 03 | bin2cell 0.3.4 (7.0M bins, A100 80GB) | yes, with 3 workarounds | load 2.5 min, StarDist 4.9 min, expand+aggregate ~3 min, ~40 GB RAM | destripe breaks on pandas 3; stardist wrapper breaks on numpy 2 + Keras 3; no stain-based false-positive check; no spillover handling |
| 03 | ENACT | not run | - | planned; same StarDist base, alternative bin assignment |
| 03 | FICTURE | not run | - | expected days on 7M bins; not feasible on Colab |
| 03 | contamination-vs-distance, registration shift test | custom | seconds-minutes | no existing tool reports spillover scale or bin-grid registration QC |
| 04 | scanpy Pearson residuals (441k bins, 2,000 HVG dense) | yes | ~1 min, 3.6 GB | NaN for zero-count units; wrapper PCA fails on NaN -> compute PCA manually |
| 04 | scVI 1.5 via scvi-colab (190k cells) | yes | 2.4 min on A100 | fine; representation-dependent clusters |
| 04 | squidpy spatial_autocorr | API changed (n_perms=0 rejected, spatial_neighbors deprecated) | - | computed Moran's I manually (kNN weights) |
| 04 | scran / sctransform | not run (R) | - | logged for R session |
| 05a | CellTypist 1.x (90k ref, 190k query) | yes (sklearn fallback) | train 8 min, predict 1.5 min | RAPIDS import breaks under pandas 3; probabilities saturate |
| 05a | scANVI joint model (ref+query, tech as batch) | ran but wrong | 7 min | over-integration: query spread over rare classes |
| 05a | scANVI reference + query mapping (scArches) | yes | 2.6 + 2.1 min | works; still 66% of CD68+ cells labelled tumor (spillover) |
| 05a | Tangram, RCTD (cell level) | not run | - | logged |
| 05b | cell2location RegressionModel (90k ref) | yes | 38 min A100 | slow (data loading); run once and cache signatures |
| 05b | cell2location spatial model (60k of 441k 8 µm bins) | yes | 16 min + 2.5 min posterior | under-trained at 2,500 epochs; full slide ~2 h; cannot separate spillover from co-occupancy |
| 05b | RCTD doublet mode (R) | not run | - | logged for R session |
| 06 | composition niches + permutation enrichment (custom) | yes | 3 min | no tool combines HD-scale composition niches with rare-niche detection |
| 06 | pybanksy 1.3.5 (190k cells, 1,000 HVG) | yes | 2.7 min | API mismatch between generate_banksy_matrix and banksy_matrix_to_adata; small island clusters need merging |
| 06 | cellcharter 0.3.7 (scVI latent, 3 layers, GMM) | yes | 5 min scan | ClusterAutoK gives no progress; manual stability scan used |
| 06 | within-tumour domains | all methods | - | no tool separates biological programs from spillover/technical gradients; needs CNV/H&E cross-check |
| 07 | infercnvpy (190k cells, 17.5k genes) | yes | 1 min (8 jobs) / 6 min (1 job) | fork pool deadlocks subsequent numba/BLAS in-session; gtfparse optional dep broken -> parse GTF manually; Arrow strings break windows |
| 07 | clone detection (PCA + k-means, silhouette) | custom | <1 min | no discrete clones; no tool separates dilution from subclonality in HD |
| 07 | CopyKAT / SCEVAN (R), Numbat / CalicoST (allele) | not run | - | R session; allele methods need HD 3' chemistry |
