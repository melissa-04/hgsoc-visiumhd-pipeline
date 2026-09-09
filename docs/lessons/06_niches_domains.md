# 06 — Niches and spatial domains (bin2cell cells, scANVI labels)

## What we did
- Composition niches: 50 µm radius neighbourhoods (median 58 neighbours), 9-type fractions, k-means k=8; permutation-based neighbourhood enrichment (20 label shuffles).
- BANKSY (pybanksy 1.3.5, lambda=0.8, 15 neighbours, 1,000 HVG): 2.7 min for 190k cells; Leiden res 0.3 -> 23 domains, 19 after merging <250-cell islands.
- CellCharter (0.3.7): scVI latent (all cells, 2.7 min) aggregated over 3 neighbour layers (120 features); manual stability scan k=6..14 x 3 seeds (5 min); k=9 (FMI 0.63).

## Key numbers
- Spatial coherence: niches 0.60, BANKSY 0.81, CellCharter 0.83.
- Composition space is ~1-D (tumour fraction ladder N5 99% -> N1 6%) plus one distinct perivascular immune niche (N7: endothelial 29%, plasma 22%, monocyte 9%).
- Enrichment: tumour vs fibroblast z=-248, fibroblast-endothelial +153, monocyte with plasma/T/mast +, monocyte vs tumour -51.
- BANKSY vs CellCharter: ARI 0.48, NMI 0.57; within tumour cells ARI 0.49. Niches vs domains ARI ~0.05 (different questions).
- CellCharter tumour sub-domains: C8 secretory core (WFDC2/SLC34A2/SERPINA5), C4 inflamed/MHC-II (CD74, LCN2, TACSTD2, 3.9% monocytes), C2 immediate-early stress (JUN/FOS/EGR1), C6 stroma-adjacent (COL1A2/FBLN1 spillover + MUC16/GREB1), C5/C0 OXPHOS/AKIRIN1/MT-ND4 with no recognisable program -> suspected regional technical gradient (same blocks seen by scVI in step 04).

## Lessons
- Composition niches over-partition a gradient; use few clusters plus explicit rare niches. Enrichment z-scores scale with edge count: read sign and rank, not magnitude.
- Expression-based domains reproduce composition niches (stroma/interface/capsule) and add within-tumour regions; every within-tumour domain must be classified as program / spillover / technical using markers, H&E and CNV.
- Domain structure here is gradient-like: stability across seeds 0.5-0.77, k choice principled but not sharp; report it as such.
- Spillover shows up as stromal genes (COL1A2, FBLN1) marking a 'tumour' domain.
- Tool notes: pybanksy 1.3 generate_banksy_matrix returns an AnnData (banksy_matrix_to_adata now breaks); cellcharter Cluster.predict returns strings; build the spatial graph yourself to avoid squidpy API drift.

## Decision
- Carry niche_comp (composition), domain_banksy_m and domain_cellcharter forward; CellCharter (k=9) as the default domain set; test C5/C0 against CNV clones in step 07.
