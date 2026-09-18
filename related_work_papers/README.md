# RouteSAM related-work paper library

Collected on 2026-09-18 for the RouteSAM paper. The library currently contains
29 validated PDF files (about 208 MB) plus three link-only records whose
publisher sites reject automated PDF downloads.

## Folder structure

| Folder | Scope | Local PDFs |
|---|---|---:|
| `01_foundation_models/` | U-Net, SAM, SAM2, SAM3, MedSAM and medical SAM benchmarks | 7 |
| `02_semi_supervised_medical_segmentation/` | Mean Teacher, pseudo supervision, consistency learning and cross-sample modeling | 9 |
| `03_sam_assisted_label_efficient/` | SAM-assisted semi-supervised and one-label medical segmentation | 5 |
| `04_cross_image_matching_and_propagation/` | In-context segmentation, support-query matching and cross-image transfer | 5 |
| `05_datasets_and_benchmarks/` | Kvasir-SEG, ISIC2018, BUSI and TN3K source papers | 3 |

Each subfolder contains a short reading guide. `references.bib` is a clean
BibTeX database for the entire collection and can later be merged into the
paper's `cas-refs.bib`.

## Recommended reading order

1. `01_foundation_models/03_SAM3_Carion_2025.pdf`
2. `04_cross_image_matching_and_propagation/04_OP_SAM_Mao_2025.pdf`
3. `04_cross_image_matching_and_propagation/02_Matcher_Liu_2023.pdf`
4. `03_sam_assisted_label_efficient/00_SemiSAM_Zhang_2023.pdf`
5. `03_sam_assisted_label_efficient/01_SemiSAM_Plus_Zhang_2025.pdf`
6. `03_sam_assisted_label_efficient/04_SC_SAM_Vu_2026.pdf`
7. `02_semi_supervised_medical_segmentation/06_DACL_Tang_2024.pdf`
8. `02_semi_supervised_medical_segmentation/07_GraphCL_Wang_2024.pdf`
9. YoloSeg (link-only record below)

This order follows the RouteSAM argument: foundation-model propagation ->
support/query matching -> generalist-specialist collaboration -> explicit
cross-sample structure -> specialist distillation for direct inference.

## Link-only papers

These papers are indexed in `references.bib`, but their publisher PDF endpoints
returned HTTP 403 during collection:

- **YoloSeg: You Only Label Once for Medical Image Segmentation** (MedIA 2026):
  https://doi.org/10.1016/j.media.2026.104093
- **SAM Foundation Model and Expert Model Cross Prompting Framework for
  Semi-Supervised Medical Image Segmentation (CPPS-SAM)** (JVCIR 2026):
  https://doi.org/10.1016/j.jvcir.2026.104876
- **Dataset of Breast Ultrasound Images (BUSI)** (Data in Brief 2020):
  https://doi.org/10.1016/j.dib.2019.104863

## Suggested Related Work organization

### Semi-supervised medical image segmentation

Use Mean Teacher, CPS, Cross Teaching, MC-Net+, BCP and DyCON to describe the
progression from consistency regularization to uncertainty-aware and
heterogeneous co-training. ACTION, DACL and GraphCL support the observation
that unlabeled images contain useful inter-sample structure.

### Foundation-model-assisted label-efficient segmentation

Use SAM/MedSAM to establish the domain gap, then SemiSAM, SemiSAM+, SAMatch,
CPC-SAM, SC-SAM, CPPS-SAM and YoloSeg to position generalist-specialist
collaboration and pseudo-label generation.

### Cross-image semantic matching and propagation

Use PerSAM, Matcher, SegGPT, UniverSeg and OP-SAM for support-query transfer.
RouteSAM differs by inserting ordered unlabeled bridge images and progressively
propagating a mask along an Anchor--Bridge--Target route, instead of performing
only direct support-to-query transfer.
