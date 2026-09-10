# hgsoc-visiumhd-pipeline

End-to-end evaluation of Visium HD analysis tools on a public high-grade serous ovarian cancer (HGSOC) section, run as preparation for a matched-cohort HGSOC organoid study (tumour, control tissue, untreated and drug-treated organoids; Visium HD + Parse snRNA-seq + WES).

The goal is not to reproduce a standard workflow but to find out **where the available tools break on HD data**, quantify it, and use those gaps to define a thesis project. Every step ends with a lessons file and an entry in the gap log.

## Data

- 10x Genomics, Visium HD Spatial Gene Expression, human ovarian cancer (papillary serous carcinoma), fresh frozen, Space Ranger 4.0.1, deep (2.3B read pairs) and minimum (441M) depth versions of the same section. CC BY 4.0.
- Reference for cell typing: MSK SPECTRUM HGSOC atlas (41 treatment-naive patients) via CELLxGENE.

## Steps

| # | Notebook | Question | Main result |
|---|---|---|---|
| 01 | `01_setup_loading` | data structure, first QC | signal leaves the tissue: edge-decay e-folding ~18 µm; bins 1–4 from the edge carry 22/13/8/5% of tissue median |
| 02 | `02_qc_artifacts` | tissue mask, local outliers, striping | stain-based masks miss RNA-rich pale tissue; fixed thresholds delete whole compartments; striping real (6× null, lag-1 −0.28) |
| 03 | `03_bin_to_cell` | Space Ranger vs bin2cell segmentation | moving from bins to cells does **not** improve purity; cross-compartment mixing ~16%, distance-dependent, symmetric — physics, not segmentation |
| 04 | `04_normalization` | log vs Pearson residuals vs scVI | log leaves depth in PC1 (r=0.61 → 0.09 with Pearson); cluster identity depends on the representation (ARI 0.32–0.43) |
| 05 | `05a_cell_typing`, `05b_bin_deconvolution` | CellTypist, scANVI, cell2location | 95.7% label agreement, but 66% of CD68+ macrophages labelled tumour; 49% of 8 µm bins are mixtures |
| 06 | `06_niches_domains` | composition niches, BANKSY, CellCharter | expression domains add within-tumour regions; one signature (AKIRIN1/NDUFS5/MACF1) has no recognisable biology |
| 07 | `07_cnv_clonality` | infercnvpy | HGSOC-consistent profile; **focal 17q12 (ERBB2/GRB7) amplification**, clonal; no discrete subclones; CNV signal diluted 25% at compartment boundaries |
| 08 | `08_cell_communication` | LIANA+ (type-level and spatial) | the top type-level pair (COL1A1–DDR1) is the bottom spatial pair; spillover creates impossible senders |
| 09 | `09_multisample` | deep vs minimum depth, pseudobulk, paired DE | depth strongly affects per-cell analysis and composition, barely affects pseudobulk; paired `~sample + condition` prototype |
| 10 | `10a_organoid_simulation`, `10b_organoid_pipeline`, `10c_correction_prototype` | organoid geometry, blind parameter estimation, correction | see below |

## Step 10: organoid simulation (no public organoid HD data exists)

A synthetic organoid slide built from the real section, with ground truth for every UMI: 24 organoids (100–300 µm), calibrated exponential PSF (λ = 4 µm), ambient floor, row/column efficiency, Poisson sampling.

- Per-organoid signal loss scales as **9 µm / diameter** (6–10% at 100 µm, 3% at 300 µm).
- **Artificial edge–core gradient**: measured/true UMI 0.70 at the edge vs 1.16 in the core, with no biological difference. Regional (edge vs core) organoid analysis sits on top of this.
- Larger bins do not help (edge/core ratio 0.41–0.56 at 2/8/16 µm).
- The AKIRIN1/NDUFS5/MACF1 signature from step 06 reappears in purely synthetic data → **technical artefact**.
- Parameters can be estimated blind: ambient within 5%, striping r = 0.87, PSF to the right order; a naive reading of the edge-decay constant overestimates the PSF by **2.3×**.
- EM/Richardson–Lucy correction: total-count correlation with truth 0.665 → **0.898**, edge gradient removed (all bands within 5% of 1.0), tolerant to a 38% PSF error. Per-cell **profiles were not improved** (0.815 → 0.781).

**Conclusion:** spillover correction splits into a *scale* problem (solved here) and a *composition* problem (not solved by cell-level EM). Tool scope v1 = per-cell scale correction, edge compensation, ambient removal, per-slide PSF/ambient/striping QC.


Large data (raw 10x outputs, processed `.h5ad`, simulation state) live on Google Drive, not in this repository.

## Environment

Google Colab (High-RAM; A100 for StarDist, scVI, cell2location). Python: scanpy, anndata, squidpy, bin2cell, StarDist, scvi-tools, CellTypist, cell2location, pybanksy, cellcharter, infercnvpy, LIANA+, PyDESeq2.

Version pitfalls encountered (all documented in `docs/gap_log.md`): pandas 3 breaks `anndata.read_10x_h5` and infercnvpy's gene windows; TensorFlow 2.20 + Keras 3 makes StarDist return zero objects; RAPIDS breaks CellTypist's import; fork-based process pools deadlock subsequent numba/BLAS calls; mid-session `pip install` corrupts pandas and torch.

## Status

Ten steps complete on public data. The pipeline and the correction prototype are ready to be applied to the project cohort when it arrives.
