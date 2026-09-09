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
