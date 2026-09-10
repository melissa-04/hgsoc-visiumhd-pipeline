# 10a — Synthetic organoid slide: simulator and ground truth

## Why simulate
No public organoid + Visium HD data exists (confirmed; the only systematic organoid ST benchmark is Stereo-seq, iScience 2026, which reports poor per-cell-bin regional signal and no improvement from larger bins). A simulator built from this real HD dataset gives what real data never can: ground truth for every UMI.

## Build
- 24 epithelial-rich circular regions (100-300 µm, >=80% tumour cells) cut from the real tumour section, 5,239 cells, 19.8M true UMI.
- Laid out on a 3,509 x 2,059 µm synthetic capture area, 4 rows with edge-to-edge spacing 120/200/300/450 µm.
- Measurement model: exponential PSF (lambda), ambient floor (0.4% of mean occupied-bin signal, tissue gene profile), row x column efficiency (log-normal, sd 0.25), Poisson sampling. Ground truth kept as a bins x cells origin matrix.

## PSF calibration (the key methodological step)
- Initial lambda = 20 µm (taken from the step-01 edge-decay length) implies only 1.5% of a cell's UMIs stay within 6 µm - inconsistent with the contamination levels measured in step 03.
- Calibrated by simulating lambda = 3, 5, 8, 12, 20 µm and matching the step-03 contamination curve (fibroblast signal in epithelial cells vs distance). lambda = 3-5 µm reproduces the measured curve at 16-48 µm; lambda >= 12 µm overshoots.
- Chosen: **lambda = 4 µm** with a 5.2 µm cell mask. Median mask purity ~0.45.
- Lesson: the observed edge-decay length is a superposition of many cells' PSFs and is not the per-cell PSF width; deconvolving one from the other requires calibration against an independent measurement.
- Lesson: a first calibration attempt failed because the metric used only 1 of 5 epithelial markers (HVG subset) and was insensitive to lambda. If a calibration says 'the parameter does not matter', suspect the metric first.

## Findings (ground truth)
- Fate of a cell's UMIs: 39.6% stay in its own mask, 60% go to neighbours inside the organoid, ~0% escape (median cell).
- Escape is confined to the outer ~10-15 µm: 32.8% of UMIs escape for cells 0-5 µm from the edge, 10.7% at 5-10 µm, 1% at 10-20 µm, 0 beyond.
- **Per-organoid signal loss scales as 9 µm / diameter**: 6-10% at 100 µm, 2.5-3.4% at 300 µm, intercept ~0. Comparing total signal across organoids of different size is therefore biased by geometry alone.
- **Artificial edge-core gradient**: measured/true UMI is 0.70 in the outer 10% of the radius, 0.86-0.90 in the middle, 1.16 in the core - a 1.6-fold range with no biological difference whatsoever. Regional (edge vs core) analysis of organoids sits on top of this physical gradient.
- Mask purity runs the other way: 0.61 at the edge vs 0.42 in the core (edge cells have fewer neighbours to contaminate them). Count-driven analyses are hurt at the edge, profile-driven analyses in the core.
- No cross-contamination between organoids at 120 µm spacing (foreign fraction 0.000 in every disc) -> **50 µm spacing is enough**; useful for slide design.
- Only 12% of the capture area is occupied (63% in the real tissue section): organoid slides provide a very large empty region for ambient/PSF estimation, which favours the correction tool.

## Environment lessons
- Building a bins x genes sparse matrix for ambient (1.8M bins x 17.6k genes) exhausts RAM; carry ambient as a per-bin count times a fixed profile instead.
- lil_matrix assembly inside a per-gene loop is unusably slow; assemble bins x cells once and multiply by a cells x genes profile matrix.
- Drive can drop mid-read on multi-GB h5ad (errno 107); copy large files to local disk first.

## Next (10b)
Run the pipeline on the synthetic slide: organoid detection, QC, cell typing, clustering; test whether the edge gradient appears as a separate cluster, how gene detection falls at the edge, and whether 2/8/16 µm bins change any of it.
