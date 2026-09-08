# 02 — QC and technical artifacts (8 µm bins)

## What we did
1. H&E-derived tissue mask (saturation, Otsu / multi-Otsu) vs Space Ranger in_tissue.
2. tissue_class labels: dense (both masks), pale (SR only), excluded.
3. Long-tail diagnosis: distance-to-tissue x distance-to-capture-edge stratification.
4. Capture-edge effect inside tissue.
5. Spatially-aware local outliers (k=24 neighbours, robust z on log UMI, log genes, %MT).
6. Striping test: residual after local smoothing, permutation null, lag-1 autocorrelation.

## Key numbers
- Single-Otsu mask missed 104,846 SR tissue bins (median 965 UMI): staining intensity != RNA presence.
- Multi-Otsu: 37,634 SR-only bins remain (median 608 UMI) -> 'pale' tissue class; HE-only 8,930 bins (median 280).
- Off-tissue bins near the capture edge (<200 µm) carry 3-4x less signal than interior bins at the same distance from tissue; long tail persists in interior (17 -> 11 -> 7 -> 4 UMI).
- Inside tissue no capture-edge depletion (dense bins near edge 2,400-2,600 vs 1,900 interior; confounded with biology in n=1 slide).
- Local outliers: 7,771 bins flagged (1.7%), scattered. Fixed threshold (<200 UMI) would drop 23,978 bins (5.4%), clustered in pale tissue/capsule. Overlap 197.
- Striping (sigma=2 bins): row-median sd 0.090 vs permutation null 0.015 (6.1x), lag-1 autocorr -0.28; columns 4.9x, -0.06. Typical row-to-row deviation ~9%, extremes +-35%.

## Lessons
- An image mask answers 'is there stain', a count mask answers 'is there RNA'. Use Space Ranger's mask as primary; keep H&E mask as dense/pale annotation.
- Pale tissue carries ~1/3 of dense counts and is real biology; label, do not drop.
- Stratify before concluding: distance-to-tissue and distance-to-capture-edge were confounded; stratification separated an edge-efficiency effect from a wide diffusion tail.
- HD QC must be local: fixed single-cell thresholds remove whole compartments and bias downstream spatial analysis.
- Artifact tests need the right expectation scale, a permutation null and an autocorrelation check; coarse smoothing (sigma=6) produced a false 'stripe' signal that was biology.
- Striping is real at 8 µm and independent between adjacent rows -> correct before cell-level analysis (bin2cell destripe at 2 µm; row x column efficiency term in the correction model).

## Open for step 03
- Run bin2cell destripe at 2 µm and compare row-effect sd before/after.
- Check whether local high-%MT flags map to cytoplasm-rich bins once cells exist.
