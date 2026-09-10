# 08 — Cell-cell communication (LIANA+ on bin2cell cells)

## What we did
- Resource: OmniPath 'consensus' (4,620 pairs). Both genes measured in the probe set: 3,061 (66%); after expression filters in the spatial run: ~490 pairs scored. Three-stage attrition documented.
- Type level: li.mt.rank_aggregate (CellPhoneDB, NATMI, Connectome, SingleCellSignalR, logFC; RobustRankAggregate), 44k balanced cells, 1,000 permutations, 0.9 min -> 8,080 interactions.
- Spatial: li.mt.bivariate, bivariate Moran's R, Gaussian weights bandwidth 30 µm (mean 31 neighbours), 60k cells, 5 min.
- Spillover test: local scores vs distance to the other compartment; type-level rerun restricted to cells >32 µm from the other compartment.

## Key numbers
- Type-level top list dominated by abundant matrix genes: COL1A1/COL3A1 -> DDR1/CD44/CD93, SPARC -> ENG. High magnitude_rank but mediocre specificity_rank.
- Same pair appears with impossible senders (Endothelial -> tumour COL1A1-DDR1), a direct signature of spillover creating false senders.
- Significant interaction counts are dominated by monocytes (1,306 cells): monocyte->monocyte 68, ->endothelial 36, ->fibroblast 34. Small, mixed populations inflate permutation significance.
- Spatial co-localisation top: SPARC-ENG 0.199, MDK-TSPAN1 0.103, LAMA5-BCAM 0.085, FN1-SDC2/ITGA9, MDK-SDC4, VIM/SPP1/FN1-CD44, SEMA4D-ERBB2.
- Spatially segregated (negative R): COL1A1-DDR1 -0.137, COL3A1-DDR1 -0.135, COL4A1-CD47, CXCL12-CXCR4 -0.050. The type-level top pair is the spatial bottom pair.
- Boundary/interior ratio of local scores splits pairs: boundary-dependent SPP1-CD44 9.2, VIM-CD44 4.3, FN1-CD44 4.1, SPARC-ENG 4.0, VWF-ITGA9 3.6, COL6A2-CD44 3.4; interior-enriched LAMA5-BCAM 0.32, SEMA4D-ERBB2 0.25, MDK-SDC4 0.35, MDK-TSPAN1 0.57.
- Interior-only rerun failed as a control: only 2,258 fibroblasts, 441 endothelial, 85 monocytes survive the >32 µm filter in this papillary tissue; rankings essentially unchanged.

## Lessons
- Type-level and spatial bivariate metrics answer different questions: the former rewards sender/receiver expression across compartments (interface pairs), the latter rewards co-location (within-compartment pairs). Report both; a pair topping one list and bottoming the other is expected, not contradictory.
- Read magnitude and specificity ranks together; magnitude alone returns collagen and SPARC in every tissue.
- Spillover produces impossible senders and inflates boundary-dependent pairs; the decay scale of those local scores (~20-30 µm) matches the spillover scale measured in step 03.
- Boundary-dependence alone cannot separate real interface signalling from spillover; a correction step (or a tissue with wide compartments) is required. In this papillary tumour almost no stromal cell is >32 µm from tumour, so the interior-only control is not usable - check control-group size before trusting a control.
- Probe-based chemistry removes a third of the LR resource before any analysis; novel-interaction claims are weak on this platform.
- Small mixed populations (monocytes, 66% of which carry tumour labels) dominate significance counts; treat rare-type CCC results as hypotheses only.

## Decision
- Carry MDK, LAMA5-BCAM, SEMA4D-ERBB2 axes as spillover-robust; flag SPP1/FN1/VIM/SPARC-CD44/ENG axes as boundary-dependent and unresolved. Use both rank_aggregate and bivariate in the project pipeline; add a corrected-vs-uncorrected comparison once the correction tool exists.
