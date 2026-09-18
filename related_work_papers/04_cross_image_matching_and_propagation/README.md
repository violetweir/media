# Cross-image matching and propagation

| File | Paper | Main concept |
|---|---|---|
| `01_PerSAM_Zhang_2023.pdf` | PerSAM | Personalizes SAM from one reference image and mask. |
| `02_Matcher_Liu_2023.pdf` | Matcher | Training-free one-shot segmentation through all-purpose feature matching. |
| `03_UniverSeg_Butoi_2023.pdf` | UniverSeg | Segments an unseen medical task from a support set without task-specific training. |
| `04_OP_SAM_Mao_2025.pdf` | OP-SAM | Transfers a one-shot polyp prior through cross-image correlations and iterative prompts. |
| `05_SegGPT_Wang_2023.pdf` | SegGPT | In-context visual prompting for general segmentation tasks. |

These methods mostly implement direct support-to-query transfer. RouteSAM's
core distinction is to organize unlabeled samples as intermediate semantic
bridges and execute progressive propagation across the resulting route.

