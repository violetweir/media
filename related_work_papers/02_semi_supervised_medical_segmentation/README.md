# Semi-supervised medical image segmentation

| File | Paper | Main concept |
|---|---|---|
| `00_Mean_Teacher_Tarvainen_2017.pdf` | Mean Teacher | Weight-averaged teacher and consistency regularization. |
| `01_Cross_Pseudo_Supervision_Chen_2021.pdf` | Cross Pseudo Supervision | Two networks supervise each other with pseudo labels. |
| `02_Cross_Teaching_CNN_Transformer_Luo_2022.pdf` | Cross Teaching between CNN and Transformer | Heterogeneous inductive biases for semi-supervised medical segmentation. |
| `03_MC_Net_Plus_Wu_2022.pdf` | MC-Net+ | Multi-decoder uncertainty and mutual consistency. |
| `04_Bidirectional_Copy_Paste_Bai_2023.pdf` | BCP | Bidirectional labeled/unlabeled mixing to reduce distribution mismatch. |
| `05_ACTION_You_2022.pdf` | ACTION | Dataset-level semantic relationships and local anatomical contrast. |
| `06_DACL_Tang_2024.pdf` | DACL | Density-aware neighbor graphs in feature space. |
| `07_GraphCL_Wang_2024.pdf` | GraphCL | Explicit graph structure across medical images. |
| `08_DyCON_Assefa_2025.pdf` | DyCON | Dynamic uncertainty-aware consistency and contrastive learning. |

For RouteSAM, the first five papers form the conventional SSL baseline family.
ACTION, DACL and GraphCL are especially important because they show that the
unlabeled pool has exploitable cross-sample geometry, although they use it for
representation learning rather than executable mask-propagation routes.

