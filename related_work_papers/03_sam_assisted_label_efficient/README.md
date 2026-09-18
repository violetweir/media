# SAM-assisted label-efficient medical segmentation

| File | Paper | Relationship to RouteSAM |
|---|---|---|
| `00_SemiSAM_Zhang_2023.pdf` | SemiSAM | Specialist-generated prompts let a frozen SAM refine supervision. |
| `01_SemiSAM_Plus_Zhang_2025.pdf` | SemiSAM+ | Generalist-specialist collaborative learning under extremely sparse labels. |
| `02_SAMatch_Xu_2024.pdf` | SAMatch | SAM-guided pseudo-label refinement inside a Match-style SSL framework. |
| `03_CPC_SAM_Miao_2024.pdf` | CPC-SAM | Cross prompting and prompt-consistency regularization. |
| `04_SC_SAM_Vu_2026.pdf` | SC-SAM | Bidirectional specialist-to-generalist adaptation and generalist-to-specialist supervision. |

Additional link-only records:

- YoloSeg: https://doi.org/10.1016/j.media.2026.104093
- CPPS-SAM: https://doi.org/10.1016/j.jvcir.2026.104876

YoloSeg is the closest precedent for training-time foundation-model propagation
followed by specialist-only inference. SemiSAM+, SC-SAM and CPPS-SAM are the
closest precedents for the planned multi-round SAM3--U-Net closed loop.

