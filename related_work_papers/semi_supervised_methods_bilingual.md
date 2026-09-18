# 半监督医学图像分割方法梳理 / Survey of Semi-Supervised Medical Image Segmentation Methods

> **面向论文 / For:** RouteSAM 相关工作、方法定位与实验 baseline 选择  
> **范围 / Scope:** 当前 LaTeX 已提到的方法、本地已收集论文，以及与 RouteSAM 最接近的 SAM/SAM2 辅助方法  
> **更新日期 / Updated:** 2026-09-18

---

## 1. 半监督分割的共同问题 / Common Problem Formulation

半监督医学图像分割使用少量有像素级标注的数据

\[
\mathcal D_L=\{(x_i,y_i)\}_{i=1}^{N_L}
\]

和大量未标注数据

\[
\mathcal D_U=\{x_j\}_{j=1}^{N_U},\qquad N_U\gg N_L,
\]

学习分割模型。大多数方法都可以写成：

\[
\mathcal L=\mathcal L_{sup}(\mathcal D_L)+\lambda(t)\mathcal L_{unsup}(\mathcal D_U),
\]

区别主要在于如何从无标注图像中构造可靠的 `L_unsup`。

Semi-supervised medical image segmentation learns from a small pixel-labeled set and a much larger unlabeled set. Most methods combine a supervised loss with a time-weighted unsupervised loss. Their central difference is how they obtain reliable supervisory signals from unlabeled images.

### 现有方法的四条主线 / Four Main Families

| 方法族 / Family | 无标注数据如何产生监督 / How unlabeled data are used | 代表方法 / Examples |
|---|---|---|
| 一致性与伪标签 / Consistency and pseudo labels | 同一图像在不同扰动、模型或分支下应产生一致预测 | Mean Teacher, CPS, Cross Teaching, MC-Net+ |
| 数据混合与不确定性 / Mixing and uncertainty | 通过 copy-paste、位移或不确定性加权控制伪标签噪声 | BCP, ABD, DyCON |
| 对比、邻域与图结构 / Contrastive, neighborhood, and graph learning | 利用数据集级相似性改善特征空间，但通常不传播 mask | ACTION, DACL, GraphCL |
| 基础模型辅助 / Foundation-model-assisted SSL | 专家模型产生提示，SAM 产生或修正伪标签；或者二者双向协同 | SemiSAM, SemiSAM+, SAMatch, CPC-SAM, SC-SAM, CPPS-SAM, YoloSeg |

---

## 2. 一致性与伪标签方法 / Consistency and Pseudo-Label Methods

### 2.1 Mean Teacher

**论文 / Paper:** [Mean Teachers Are Better Role Models](https://arxiv.org/abs/1703.01780)  
**本地 PDF / Local PDF:** [`00_Mean_Teacher_Tarvainen_2017.pdf`](02_semi_supervised_medical_segmentation/00_Mean_Teacher_Tarvainen_2017.pdf)

**怎么做 / How it works**

- 使用结构相同的 Student 和 Teacher。
- Student 通过反向传播更新；Teacher 不反向传播，而是 Student 权重的指数滑动平均：

\[
\theta_T\leftarrow \alpha\theta_T+(1-\alpha)\theta_S.
\]

- 标注图像使用真实 mask 监督 Student。
- 未标注图像经过不同噪声或增强后分别送入 Student 和 Teacher，最小化两者预测的一致性损失。

The student is optimized by gradient descent, while the teacher is an exponential moving average of student weights. Labeled samples use ground-truth supervision, and unlabeled samples enforce consistency between perturbed student and teacher predictions.

**优点 / Strengths:** 简单、稳定、通用，是医学半监督分割最常见的基础框架。  
**局限 / Limitations:** Teacher 与 Student 高度同源，错误可能被 EMA 稳定下来并形成 confirmation bias；没有显式利用不同图像之间的关系。  
**推理 / Inference:** 通常只保留 Teacher 或 Student 单模型，直接单图推理。  
**与 RouteSAM 的区别 / Difference from RouteSAM:** Mean Teacher 只约束同一图像在扰动下的一致性，RouteSAM 则显式选择其他图像作为 Anchor 和 Bridge 来传播监督。

### 2.2 Cross Pseudo Supervision (CPS)

**论文 / Paper:** [Semi-Supervised Semantic Segmentation with Cross Pseudo Supervision](https://openaccess.thecvf.com/content/CVPR2021/html/Chen_Semi-Supervised_Semantic_Segmentation_With_Cross_Pseudo_Supervision_CVPR_2021_paper.html)  
**本地 PDF / Local PDF:** [`01_Cross_Pseudo_Supervision_Chen_2021.pdf`](02_semi_supervised_medical_segmentation/01_Cross_Pseudo_Supervision_Chen_2021.pdf)

**怎么做 / How it works**

- 使用两个结构相同但随机初始化不同的分割网络。
- 对同一输入，网络 A 的 argmax one-hot 预测作为网络 B 的伪标签，网络 B 反过来监督网络 A。
- 两个网络都参与反向传播；监督损失与双向 cross-pseudo loss 联合训练。

Two independently initialized segmentation networks generate hard pseudo labels for each other. Both networks are updated, combining labeled supervision with bidirectional cross-pseudo supervision on unlabeled inputs.

**优点:** 实现简单；网络扰动比单模型自训练提供更多预测多样性。  
**局限:** 两个网络可能逐渐耦合并共同确认错误；原始 CPS 是自然图像语义分割方法，并非专为医学图像设计。  
**推理:** 使用一个网络或两个网络集成。  
**与 RouteSAM 的区别:** CPS 的监督在同一图像的两个模型之间交换，不涉及跨图像检索或渐进传播。

### 2.3 Cross Teaching between CNN and Transformer

**论文 / Paper:** [Semi-Supervised Medical Image Segmentation via Cross Teaching between CNN and Transformer](https://proceedings.mlr.press/v172/luo22b.html)  
**本地 PDF / Local PDF:** [`02_Cross_Teaching_CNN_Transformer_Luo_2022.pdf`](02_semi_supervised_medical_segmentation/02_Cross_Teaching_CNN_Transformer_Luo_2022.pdf)

**怎么做 / How it works**

- 使用 CNN 和 Transformer 两个异构分割网络。
- 两者分别由真实标签训练，同时将自己的预测伪标签交给另一网络。
- 利用 CNN 的局部归纳偏置和 Transformer 的全局建模能力形成互补。

A CNN and a Transformer directly teach each other using pseudo labels. Their heterogeneous inductive biases provide more meaningful diversity than two identically structured networks.

**优点:** 结构异构能降低同构双网络的错误相关性。  
**局限:** 训练成本较高；Transformer 在极低标注预算下可能冷启动困难。  
**推理:** 可选择验证集更优的单分支，或进行双模型集成。  
**与 RouteSAM 的区别:** 互补来自模型架构，而 RouteSAM 的互补来自跨图像语义路线及 generalist-specialist 分工。

### 2.4 MC-Net+

**论文 / Paper:** [Mutual Consistency Learning for Semi-Supervised Medical Image Segmentation](https://www.sciencedirect.com/science/article/pii/S1361841522001773)  
**本地 PDF / Local PDF:** [`03_MC_Net_Plus_Wu_2022.pdf`](02_semi_supervised_medical_segmentation/03_MC_Net_Plus_Wu_2022.pdf)

**怎么做 / How it works**

- 一个共享 Encoder 连接多个略有差异的 Decoder，主要差异来自上采样方式。
- 多 Decoder 输出的统计差异作为一次前向传播内的像素/体素不确定性估计。
- 每个 Decoder 的概率输出与其他 Decoder 经过 sharpening 的软伪标签进行 mutual consistency 学习。
- 重点推动模型在粘连边界、细分支等高不确定区域得到一致、低熵的预测。

MC-Net+ uses one shared encoder and multiple slightly different decoders. Decoder disagreement estimates uncertainty, while mutual consistency between probability outputs and sharpened soft pseudo labels regularizes difficult regions.

**优点:** 不需要多次 Monte Carlo 推理即可估计不确定性；推理可回到普通 encoder-decoder。  
**局限:** 多 Decoder 增加训练显存；分支仍共享 Encoder，错误并不完全独立。  
**与 RouteSAM 的关系:** 可作为 RouteSAM 最终 U-Net specialist 的更强替代或不确定性审核器，但它本身不做跨样本监督传播。

---

## 3. 数据混合与不确定性方法 / Data Mixing and Uncertainty-Aware Methods

### 3.1 Bidirectional Copy-Paste (BCP)

**论文 / Paper:** [Bidirectional Copy-Paste for Semi-Supervised Medical Image Segmentation](https://openaccess.thecvf.com/content/CVPR2023/html/Bai_Bidirectional_Copy-Paste_for_Semi-Supervised_Medical_Image_Segmentation_CVPR_2023_paper.html)  
**本地 PDF / Local PDF:** [`04_Bidirectional_Copy_Paste_Bai_2023.pdf`](02_semi_supervised_medical_segmentation/04_Bidirectional_Copy_Paste_Bai_2023.pdf)

**怎么做 / How it works**

- 以 Mean Teacher 为基础。
- 将标注图像的随机区域粘贴到未标注背景，同时也将未标注区域粘贴到标注背景，形成双向混合。
- 混合图像由 Student 预测；监督 mask 由真实标签和 Teacher 伪标签按同一空间区域拼接。
- 目标是缓解少量标注集和大量未标注集之间的经验分布偏差。

BCP performs bidirectional region mixing between labeled and unlabeled images in a Mean Teacher framework. The supervisory target is mixed from ground truth and teacher pseudo labels using the same spatial mask.

**优点:** 概念清晰、代码成熟、通常是强医学 SSL baseline。  
**局限:** copy-paste 可能破坏病灶与背景的真实解剖关系；仍依赖 Teacher 伪标签。  
**推理:** 单模型直接推理。  
**与 RouteSAM 的区别:** BCP 在像素空间混合两个样本；RouteSAM 在语义空间组织多个完整图像并沿路线传播 mask。

### 3.2 Adaptive Bidirectional Displacement (ABD)

**论文 / Paper:** [Adaptive Bidirectional Displacement for Semi-Supervised Medical Image Segmentation](https://openaccess.thecvf.com/content/CVPR2024/html/Chi_Adaptive_Bidirectional_Displacement_for_Semi-Supervised_Medical_Image_Segmentation_CVPR_2024_paper.html)

**怎么做 / How it works**

- 针对多种扰动叠加后难以控制伪标签质量的问题，依据预测置信度进行 patch 位移。
- 对未标注图像，利用可靠区域进行双向位移，抑制不可控内容同时保留扰动作用。
- 对标注图像，使用反向置信度将更不可靠的内容引入训练，迫使模型学习困难区域。

ABD uses prediction confidence to control bidirectional patch displacement. Reliable regions guide perturbations on unlabeled images, while inverse-confidence displacement on labeled images deliberately exposes the model to difficult content.

**优点:** 比固定 CutMix/Copy-Paste 更关注区域可靠性；可叠加在不同 consistency baseline 上。  
**局限:** 置信度不一定等于真实正确率；实现和超参数比 BCP 更复杂。  
**与 RouteSAM 的关系:** ABD 是合适的强传统 baseline，也可作为 RouteSAM specialist 训练阶段的数据增强模块，但不应与路线贡献混为一体。

### 3.3 DyCON

**论文 / Paper:** [DyCON: Dynamic Uncertainty-aware Consistency and Contrastive Learning](https://openaccess.thecvf.com/content/CVPR2025/html/Assefa_DyCON_Dynamic_Uncertainty-aware_Consistency_and_Contrastive_Learning_for_Semi-supervised_Medical_CVPR_2025_paper.html)  
**本地 PDF / Local PDF:** [`08_DyCON_Assefa_2025.pdf`](02_semi_supervised_medical_segmentation/08_DyCON_Assefa_2025.pdf)

**怎么做 / How it works**

- `UnCL`：按体素熵动态加权一致性损失，不直接丢弃高不确定区域；训练早期以较小惩罚探索困难区域，后期逐渐转向稳定的高置信预测。
- `FeCL`：在 patch-level contrastive learning 中加入双 focal 机制与自适应置信度，强调 hard positive、hard negative 和不确定样本对。
- 同时处理 3D 病灶分割中的类别不平衡、边界不确定和局部特征区分问题。

DyCON combines an uncertainty-aware consistency loss with a focal entropy-aware contrastive loss. It dynamically reweights voxels instead of discarding uncertain regions and emphasizes hard, uncertain contrastive pairs.

**优点:** 对小病灶、类别不平衡和不规则边界有针对性。  
**局限:** 原实验主要是 3D ISLES、BraTS、LA、Pancreas；迁移到 RouteSAM 的 2D RGB/超声数据需要重新调参。  
**与 RouteSAM 的关系:** 适合在讨论中作为最新传统 SSL 代表，但未必是前三个数据集上最公平、最省成本的主 baseline。

---

## 4. 跨样本表征学习 / Cross-Sample Representation Learning

这类方法与 RouteSAM 共享一个重要观察：未标注图像并非彼此独立，数据集内部的邻域、聚类与解剖关系可以提供监督。然而，它们主要改变**特征学习目标**，通常不把跨样本关系变成可执行的 mask 传播路径。

These methods share RouteSAM's observation that unlabeled images are not independent. However, they mainly regularize representation geometry rather than converting cross-sample relations into executable mask-propagation routes.

### 4.1 ACTION

**论文 / Paper:** [Bootstrapping Semi-Supervised Medical Image Segmentation with Anatomical-Aware Contrastive Distillation](https://arxiv.org/abs/2206.02307)  
**本地 PDF / Local PDF:** [`05_ACTION_You_2022.pdf`](02_semi_supervised_medical_segmentation/05_ACTION_You_2022.pdf)

**怎么做 / How it works**

- 建立 BYOL 风格的 Student/EMA Teacher。
- 第一阶段进行全局 contrastive distillation；第二阶段进行局部 contrastive distillation；第三阶段进行 anatomical contrast fine-tuning。
- 对负样本使用软关系而不是简单二值正负定义，以表达数据集级语义相似性。
- 主动采样困难负像素，进行类别级 anatomical contrast，改善类别不平衡和边界。

ACTION performs global and local contrastive distillation followed by anatomical contrast fine-tuning. It models soft semantic relations among negatives and actively samples difficult pixels to improve class balance and boundaries.

**与 RouteSAM 的关键差别:** ACTION 将跨图像关系用于特征蒸馏；RouteSAM 用它来决定监督从哪张 Anchor 经过哪些 Bridge 到达 Target。

### 4.2 Density-Aware Contrastive Learning (DACL)

**论文 / Paper:** [Neighbor Does Matter: Density-Aware Contrastive Learning](https://arxiv.org/abs/2412.19871)  
**本地 PDF / Local PDF:** [`06_DACL_Tang_2024.pdf`](02_semi_supervised_medical_segmentation/06_DACL_Tang_2024.pdf)

**怎么做 / How it works**

- 在 co-teaching 框架上构建 labeled+unlabeled 的 density-aware neighbor graph。
- 根据局部近邻平均距离估计特征密度，在类簇稀疏区域寻找 Anchor feature，在高密度区域寻找近似 cluster center 的 positive key。
- 将稀疏 Anchor 拉向高密度正样本，使类内特征更紧致。
- 结合 label-guided cross supervision 与 density-guided geometric regularization。

DACL estimates local feature density using neighbor graphs. Sparse-region anchor features are pulled toward high-density positives approximating class centers, while co-teaching supplies complementary pseudo supervision.

**与 RouteSAM 的关键差别:** DACL 的 “anchor” 是特征对比学习中的 anchor feature，不是人工标注的传播起点；它不生成有序的图像路线。

### 4.3 GraphCL

**论文 / Paper:** [GraphCL: Graph-Based Clustering for Semi-Supervised Medical Image Segmentation](https://arxiv.org/abs/2411.13147)  
**本地 PDF / Local PDF:** [`07_GraphCL_Wang_2024.pdf`](02_semi_supervised_medical_segmentation/07_GraphCL_Wang_2024.pdf)

**怎么做 / How it works**

- 由 CNN 特征构造 dense instance graph，每个节点对应样本或局部特征。
- GCN 通过可学习加权边传播结构信息。
- 将局部特征 pairwise affinity 与原始特征结合，用 k-less clustering 自动形成簇，而不预先规定簇数。
- 与 teacher-student 和 BCP 式训练结合，改善跨图结构的聚类特征。

GraphCL constructs a dense instance graph from CNN features, propagates structural information through a GCN, and performs k-less clustering using pairwise affinities and raw features.

**与 RouteSAM 的关键差别:** GraphCL 的边服务于隐空间聚类和表征更新；RouteSAM 的边决定真实的跨图像 SAM3 传播顺序，并最终产生 mask。

---

## 5. SAM 与基础模型辅助半监督 / SAM-Assisted Semi-Supervised Learning

### 5.1 SemiSAM

**论文 / Paper:** [SemiSAM: Enhancing Semi-Supervised Medical Image Segmentation via SAM-Assisted Consistency Regularization](https://arxiv.org/abs/2312.06316)  
**正式版本 / Published version:** BIBM 2024, DOI `10.1109/BIBM62325.2024.10821951`  
**本地 PDF / Local PDF:** [`00_SemiSAM_Zhang_2023.pdf`](03_sam_assisted_label_efficient/00_SemiSAM_Zhang_2023.pdf)

**怎么做 / How it works**

- 在现有 SSL 框架（例如 Mean Teacher）旁增加一个 SAM-assisted branch。
- 任务分割模型先对未标注图像产生粗分割和不确定性。
- 从低不确定区域抽取点提示并输入 SAM/SAM-Med3D。
- SAM 输出作为额外伪标签，与主模型预测计算 SAM consistency loss。
- SAM 的权重通常保持冻结；其监督权重在训练早期更高，后期逐渐减弱，以降低错误提示的影响。

SemiSAM augments a conventional SSL model with a frozen SAM branch. The task model localizes the object and generates reliable prompts; SAM returns pseudo masks that provide an additional consistency target.

**优点:** 插件式加入现有 SSL；只在训练期需要基础模型。  
**局限:** 提示质量完全依赖 specialist 的早期粗预测；SAM 不适配目标域时可能持续产生有偏伪标签。  
**推理:** 只使用 task-specific specialist，自动单图推理。  
**与 RouteSAM 的区别:** SemiSAM 对每张未标注图像独立调用 SAM，不使用其他未标注图像作为 Bridge。

### 5.2 SemiSAM+

**论文 / Paper:** [SemiSAM+: Rethinking Semi-Supervised Medical Image Segmentation in the Era of Foundation Models](https://doi.org/10.1016/j.media.2025.103733)  
**本地 PDF / Local PDF:** [`01_SemiSAM_Plus_Zhang_2025.pdf`](03_sam_assisted_label_efficient/01_SemiSAM_Plus_Zhang_2025.pdf)

**怎么做 / How it works**

- 一个可训练的 task-specific specialist 搭配一个或多个冻结的 promptable generalist。
- specialist 在标注集上使用监督损失，在未标注集上保留原 SSL 正则。
- specialist 的预测转换为点或 mask 等位置提示，送给 generalist。
- generalist 生成伪标签，通过 confidence-aware regularization 反向监督 specialist。
- specialist 的改进又提升后续自动提示质量，形成训练期协作循环。

SemiSAM+ couples a trainable specialist with one or more frozen promptable generalists. The specialist generates prompts, the generalists return pseudo labels, and confidence-aware regularization transfers their pretrained knowledge back to the specialist.

**优点:** generalist 可插拔、多 generalist 可融合；推理只保留轻量 specialist，无额外开销。  
**局限:** 论文实验主要是 3D 任务；generalist 冻结，不能真正吸收目标域知识；协作发生在同一图像内。  
**与 RouteSAM 的区别:** 两者都采用 generalist-specialist 叙事，但 RouteSAM 还显式构造跨图像路线，并计划让 specialist 进一步指导 SAM3 适配与重传播。

### 5.3 SAMatch

**论文 / Paper:** [A SAM-Guided and Match-Based Semi-Supervised Segmentation Framework for Medical Imaging](https://arxiv.org/abs/2411.16949)  
**本地 PDF / Local PDF:** [`02_SAMatch_Xu_2024.pdf`](03_sam_assisted_label_efficient/02_SAMatch_Xu_2024.pdf)

**怎么做 / How it works**

- 以 FixMatch/Mean-Teacher 风格的 weak-to-strong Match 框架为主体。
- Teacher 对弱增强图像输出预测，自动提取高置信点或 box，作为 SAM 的提示。
- 经过目标任务微调的 SAM 根据提示生成更完整的 mask。
- SAM mask 替代普通 Teacher 预测，作为强增强 Student 的高质量伪标签。
- Match 模型训练与 SAM 微调协同进行，二者动态改善。

SAMatch lets the Match-based teacher generate automatic prompts for a task-adapted SAM. SAM then returns refined masks as pseudo labels for the strongly augmented student, while the Match model and SAM are optimized synergistically.

**优点:** 同时解决 Match 伪标签粗糙和 SAM 需要人工提示的问题；论文直接包含 BUSI。  
**局限:** 双模型联合训练开销大；SAM 伪标签仍依赖当前 Teacher 提示，错误可能循环放大。  
**推理:** 主要使用自动 Match 分割模型；若使用 SAM 分支则仍需要自动提示。  
**与 RouteSAM 的区别:** SAMatch 是同一图像内的 weak/strong consistency 与提示闭环，没有从其他图像逐步转移监督。

### 5.4 CPC-SAM

**论文 / Paper:** [Cross Prompting Consistency with Segment Anything Model for Semi-Supervised Medical Image Segmentation](https://papers.miccai.org/miccai-2024/170-Paper0321.html)  
**本地 PDF / Local PDF:** [`03_CPC_SAM_Miao_2024.pdf`](03_sam_assisted_label_efficient/03_CPC_SAM_Miao_2024.pdf)

**怎么做 / How it works**

- 将 SAM 改为共享 image encoder 和 prompt encoder、但拥有两个不同初始化 mask decoder 的双分支网络。
- 分支 A 的无提示输出经最大连通区域转换成点提示，提示分支 B；B 的 prompted prediction 监督 A 的 unprompted prediction，反向同理。
- Prompt Consistency Regularization 约束不同提示位置下输出不变，降低 SAM 对点位置的敏感性。
- SAM 在有标注和无标注数据上共同 fine-tune；推理时使用学习到的默认 dense prompt embedding，实现自动分割。

CPC-SAM fine-tunes a dual-decoder SAM. Each branch converts its unprompted output into a point prompt for the other branch, whose prompted mask supervises the first. Prompt-consistency regularization reduces sensitivity to prompt locations.

**优点:** 直接让 SAM 吸收未标注目标域数据；BUSI 是其核心实验之一；推理无需人工提示。  
**局限:** 两个 SAM Decoder 仍可能耦合；训练显存较高；没有 task-specific CNN specialist。  
**与 RouteSAM 的区别:** CPC-SAM 的“cross”是同图双分支交叉提示，RouteSAM 的“cross-image”是真正跨样本传播。

### 5.5 SC-SAM

**论文 / Paper:** [From Specialist to Generalist: Unlocking SAM's Learning Potential on Unlabeled Medical Images](https://arxiv.org/abs/2601.17934)  
**本地 PDF / Local PDF:** [`04_SC_SAM_Vu_2026.pdf`](03_sam_assisted_label_efficient/04_SC_SAM_Vu_2026.pdf)

**怎么做 / How it works**

- U-Net specialist 通过常规半监督策略先学习目标域结构。
- U-Net 预测被转换成点提示和伪标签，指导 SAM 的 parameter-efficient fine-tuning。
- 适配中的 SAM 反过来生成更精细的 mask，正则和稳定 U-Net。
- 两个模型形成双向 co-training：specialist→generalist 提供域结构，generalist→specialist 提供预训练语义先验。

SC-SAM forms a bidirectional co-training loop. A U-Net supplies point prompts and pseudo labels for parameter-efficient SAM adaptation, while SAM produces refined masks that regularize the U-Net.

**优点:** 真正让 specialist 帮助 generalist 使用无标注数据；包含 polyp 跨数据集实验，与 Kvasir 相关。  
**局限:** 2026 年较新，复现成熟度和公平协议需单独检查；协同依然以单张图像为单位。  
**与 RouteSAM 的区别:** SC-SAM 是最接近 RouteSAM 多轮 generalist-specialist 闭环的工作，但 RouteSAM 额外规定了 Anchor 来源、Bridge 顺序和冻结路线上的重传播。

### 5.6 CPPS-SAM

**论文 / Paper:** [SAM Foundation Model and Expert Model Cross Prompting Framework for Semi-Supervised Medical Image Segmentation](https://doi.org/10.1016/j.jvcir.2026.104876)

**怎么做 / How it works**

- 将 ViT-based SAM generalist 与 CNN expert 结合。
- 两个异构模型通过 cross prompting 和 cross pseudo supervision 双向传递知识。
- CNN 提供任务相关结构与提示，SAM 提供预训练先验和伪监督。
- SAM 的 unprompted output branch 支持测试时自动、无提示分割。

CPPS-SAM combines a ViT-based SAM and a CNN expert through cross prompting and cross pseudo supervision. Its unprompted SAM output branch supports automatic prompt-free inference.

**优点:** 同时利用网络异构性、交叉提示与双向伪监督。  
**局限:** 2026 年新方法，当前本地尚无全文 PDF；需要根据正式代码进一步核对训练细节和资源需求。  
**与 RouteSAM 的区别:** CPPS-SAM 解决同图模型间的双向知识传递，而 RouteSAM 研究监督在多张图像之间应按什么顺序传播。

---

## 6. 单标注与跨图像传播方法 / One-Shot and Cross-Image Propagation

这类方法不一定属于标准的半监督训练范式，但与 RouteSAM 的问题设置最接近，因为它们明确从 support/labeled image 向其他图像转移 mask 信息。

These methods are not always conventional SSL, but they are conceptually close to RouteSAM because they explicitly transfer mask information from a support image to other images.

### 6.1 OP-SAM

**论文 / Paper:** [One Polyp Identifies All: One-Shot Polyp Segmentation with SAM via Cascaded Priors and Iterative Prompt Evolution](https://arxiv.org/abs/2507.16337)  
**本地 PDF / Local PDF:** [`04_OP_SAM_Mao_2025.pdf`](04_cross_image_matching_and_propagation/04_OP_SAM_Mao_2025.pdf)

**怎么做 / How it works**

- 从单张 support image 与 mask 出发。
- Correlation-based Prior Generation 在 support-query 特征间做相关性匹配，生成 query 的语义先验。
- Scale-cascaded Prior Fusion 处理息肉尺度变化并过滤噪声转移。
- Euclidean Prompt Evolution 不一次性输入所有点，而是迭代添加/更新提示以逐步改善 SAM 输出。

OP-SAM transfers a semantic prior from one annotated support image to a query through feature correlation, fuses scale-cascaded priors, and iteratively evolves prompts for SAM.

**局限与区别:** 它仍是 support→query 的直接传递，每个 query 在推理时需要 support correlation 与提示演化；RouteSAM 使用未标注 Bridge 分解较大的语义跨度，并计划将传播知识蒸馏到最终单图模型。

### 6.2 YoloSeg

**论文 / Paper:** [YoloSeg: You Only Label Once for Medical Image Segmentation](https://doi.org/10.1016/j.media.2026.104093)  
**代码 / Code:** [Official YoloSeg repository](https://github.com/iMED-Lab/YoloSeg)

**怎么做 / How it works**

YoloSeg 是两阶段框架：

1. 从一张已标注图像出发，用 SAM2 向未标注数据传播标签，生成多视角伪标签；
2. 将多视角输出分为共识区域和分歧区域；
3. 使用 dual-component loss 分别学习可靠共识和不可靠分歧；
4. 使用 cross-patch augmentation 构造语义更一致的训练样本；
5. 最终训练普通 segmentation specialist，测试时不再需要 SAM2 传播。

YoloSeg first uses SAM2 to propagate one labeled mask to unlabeled images under multiple views. It decomposes predictions into consensus and divergence regions, trains a specialist with a dual-component loss, and uses cross-patch augmentation for robust pseudo-label learning.

**优点:** 只需一张标注图；明确处理传播噪声；最终 specialist 单图推理；在十个数据集上验证。  
**局限:** 传播本身仍是从单个标注源直接扩散，主要通过多视角一致性处理噪声，没有显式的 Anchor 自动选择与多 Bridge 路线。  
**与 RouteSAM 的关系:** 这是 RouteSAM 最重要的直接竞争方法。两者都使用训练期基础模型传播并在测试时保留 specialist；RouteSAM 的独特性必须落在“覆盖感知 Anchor + 有序 Bridge 路线 + 路线审核/重传播”上。

### 6.3 PerSAM、Matcher、UniverSeg 与 SegGPT

| Method | 核心机制 / Core mechanism | 为什么不是标准 SSL / Why not standard SSL | 对 RouteSAM 的作用 / Relevance |
|---|---|---|---|
| [PerSAM](https://arxiv.org/abs/2305.03048) | 用一张参考图和 mask 个性化 SAM | 主要是 one-shot personalization | reference-conditioned baseline |
| [Matcher](https://arxiv.org/abs/2305.13310) | 通用特征匹配产生 SAM 提示，training-free one-shot segmentation | 不联合训练 labeled/unlabeled pool | cross-image matching baseline |
| [UniverSeg](https://openaccess.thecvf.com/content/ICCV2023/html/Butoi_UniverSeg_Universal_Medical_Image_Segmentation_ICCV_2023_paper.html) | 给定 support set，在未见任务上直接分割 query | 属于 universal/few-shot segmentation | support-set inference baseline |
| [SegGPT](https://openaccess.thecvf.com/content/ICCV2023/html/Wang_SegGPT_Segmenting_Everything_in_Context_ICCV_2023_paper.html) | 将图像和 mask 作为视觉上下文提示 | in-context segmentation，不是常规 SSL | general in-context comparison |

这些方法可以放 Related Work，但不应与 Mean Teacher、BCP 等使用相同的“半监督训练 baseline”标签。

These methods belong in Related Work, but should not be presented as conventional semi-supervised training baselines.

---

## 7. 方法对比矩阵 / Method Comparison Matrix

| Method | 无标注监督来源 / Unlabeled supervision | 跨图像关系 / Cross-image relation | Foundation model | FM 是否训练 / FM adapted? | 测试时是否需提示或参考图 / Test-time prompt or support | 与 RouteSAM 最主要差异 / Main difference |
|---|---|---|---|---|---|---|
| Mean Teacher | EMA Teacher prediction | no | no | - | no | only perturbation consistency |
| CPS | peer-network hard pseudo label | no | no | - | no | same-image cross-network teaching |
| Cross Teaching | CNN/Transformer mutual pseudo labels | no | no | - | no | heterogeneity comes from architecture |
| MC-Net+ | multi-decoder soft pseudo labels | no | no | - | no | uncertainty from decoder disagreement |
| BCP | Teacher pseudo label + GT mixing | pairwise pixel mixing | no | - | no | mixes pixels, not semantic routes |
| ABD | confidence-guided displacement | pairwise patch mixing | no | - | no | perturbation control, no propagation |
| DyCON | uncertainty-weighted consistency + contrast | local feature pairs | no | - | no | learns difficult regions, not routes |
| ACTION | teacher contrastive targets | dataset/global-local similarity | no | - | no | relation used for representation learning |
| DACL | density neighbor graph + co-teaching | feature-neighborhood graph | no | - | no | graph regularizes features, not masks |
| GraphCL | instance graph and GCN clustering | explicit feature graph | no | - | no | graph edges are latent representation edges |
| SemiSAM | SAM pseudo mask | no | SAM/SAM-Med3D | frozen | no, specialist only | per-image SAM supervision |
| SemiSAM+ | one/multiple generalist pseudo masks | no | promptable generalists | frozen | no, specialist only | same-image specialist→generalist prompts |
| SAMatch | fine-tuned SAM pseudo mask | no | SAM/MedSAM variants | yes | normally no for Match model | weak/strong matching within one image |
| CPC-SAM | dual-SAM cross-prompt target | no | SAM | yes | no, learned default prompt | cross-branch, not cross-image |
| SC-SAM | reciprocal U-Net/SAM supervision | no | SAM | PEFT | automatic branch | bidirectional co-training without routes |
| CPPS-SAM | CNN-SAM cross prompting/pseudo labels | no | SAM | yes | no, unprompted branch | model-level cross prompting |
| OP-SAM | support-query prior and iterative prompts | direct support→query | SAM | mainly prompt evolution | **yes** | no unlabeled Bridge chain |
| YoloSeg | SAM2 multi-view propagation labels | direct labeled→unlabeled | SAM2 | propagation engine | no, final specialist | no automatic Anchor or ordered Bridges |
| **RouteSAM** | route-propagated multi-candidate masks + specialist audit | **ordered Anchor→Bridges→Target** | SAM3 | planned adaptation | no, final specialist | explicit source selection and propagation path |

---

## 8. RouteSAM 的论文定位 / Positioning RouteSAM

### 现有工作已经解决什么 / What existing work already solves

1. Mean Teacher、CPS、Cross Teaching 证明了无标注图像上的模型一致性可以提供训练信号。
2. BCP、ABD、DyCON 主要解决伪标签噪声、经验分布偏差和困难区域学习。
3. ACTION、DACL、GraphCL 证明数据集内部存在可利用的跨样本语义结构。
4. SemiSAM、SemiSAM+、SAMatch、CPC-SAM、SC-SAM 和 CPPS-SAM 证明 foundation model 与 specialist 可以互相生成提示或伪监督。
5. OP-SAM 和 YoloSeg 证明标注 mask 可以借助 SAM/SAM2 向其他图像转移。

### RouteSAM 需要保住的独特问题 / The distinct question RouteSAM must preserve

> 现有方法大多研究“如何让同一张未标注图像获得更可靠的预测”，或者“如何从一个标注 support 直接转移到 query”。RouteSAM 研究的是：当标注起点与 Target 的语义跨度过大时，能否从无标注池中选择一系列 Bridge，把一次困难的直接传递分解为多次较平滑的跨图像传播。

> Existing methods mostly ask how to obtain a more reliable prediction for each unlabeled image independently, or how to transfer directly from one labeled support to a query. RouteSAM instead asks whether a difficult source-to-target transfer can be decomposed into smoother cross-image transitions through selected unlabeled Bridges.

因此 RouteSAM 的贡献应按以下层次表达：

1. **Propagation source selection:** 在固定标注预算下选择覆盖面更广的 Anchor；
2. **Propagation path construction:** 不只匹配 Anchor 与 Target，还选择并排序 Bridge；
3. **Path execution:** SAM3 沿有序路线逐步传播，而不是仅对 Target 独立生成伪标签；
4. **Path-aware quality control:** 对多个路线候选进行校准、排序和 specialist 审核；
5. **Knowledge absorption:** 最终将路线监督吸收到可直接单图推理的 specialist 中。

---

## 9. 建议的正式 Baseline / Recommended Experimental Baselines

### A 级：主表应尽量包含 / Tier A: Strongly Recommended for the Main Table

| Baseline | 选择理由 / Why include it | 实现建议 / Implementation note |
|---|---|---|
| 1% supervised U-Net | 标注预算下限；验证无标注数据是否真正有用 | 与 RouteSAM 使用完全相同的 Anchor GT |
| Mean Teacher | 最标准 teacher-student baseline | 统一 U-Net、增强和训练步数 |
| CPS | 最标准 cross-pseudo baseline | 两个相同 U-Net；报告单模型/集成规则 |
| BCP | 强医学 SSL baseline | 将 3D patch mixing 改为 2D 时固定规则 |
| ABD | 较新的强 perturbation baseline | 使用官方实现并控制输入尺寸和 backbone |
| SemiSAM+ 或 SemiSAM | 训练期 foundation-model teacher | 更推荐选与当前任务维度一致的版本 |
| SAMatch | 与 BUSI 直接相关，代表 SAM-refined pseudo labels | Kvasir/ISIC 需重新适配提示规则 |
| CPC-SAM | SAM 本身利用无标注数据并支持无提示推理 | BUSI 上尤其重要 |
| YoloSeg | 与极低标注和 propagation 最接近 | 若按 1% 多 Anchor，需同时报告原 one-label 与公平预算版本 |

### B 级：适合补充表或定性比较 / Tier B: Supplementary or Conceptual Comparisons

- MC-Net+：代表多 Decoder 不确定性；
- Cross Teaching：代表 CNN-Transformer 异构互教；
- DyCON：代表 2025 年不确定性+对比学习；
- ACTION、DACL、GraphCL：用于跨样本关系的相关工作和表征消融；
- OP-SAM：用于 Kvasir 的直接 support→query 对照；
- SC-SAM：用于 Kvasir/polyp 的 specialist-generalist 双向闭环对照；
- CPPS-SAM：若正式代码可复现，可加入最新 SAM-expert 双向协同对照。

### 数据集相关优先级 / Dataset-Specific Priority

| Dataset | 优先比较 / Highest-priority comparisons |
|---|---|
| Kvasir-SEG | Mean Teacher, CPS, BCP/ABD, SemiSAM+, SC-SAM, OP-SAM, YoloSeg |
| BUSI | Mean Teacher, BCP/ABD, SAMatch, CPC-SAM, YoloSeg |
| ISIC2018 | Mean Teacher, CPS, BCP/ABD, SAMatch, YoloSeg |

---

## 10. 公平比较时必须统一的设置 / Requirements for Fair Comparison

1. **相同标注样本。** 所有 baseline 必须使用同一批自动选择 Anchor，另做随机 Anchor 消融。  
   **Same labeled samples:** every baseline must use the same selected Anchors.

2. **相同标注预算定义。** 明确 1% 是按图像数、病例数还是切片数计算。  
   **Same budget definition:** image-, case-, or slice-level labeling must be explicit.

3. **相同数据划分与预处理。** 特别避免不同论文的官方 split 与当前 split 混用。  
   **Same splits and preprocessing.**

4. **验证集选模型，test 只做冻结评估。** 禁止根据 test 选择 checkpoint、阈值或路线。  
   **Validation-only model selection; frozen test evaluation.**

5. **区分训练期和测试期开销。** 报告 SAM/SAM2/SAM3 是否只在训练期使用，测试时是否需要 support、检索或 prompt。  
   **Separate training-time and test-time costs.**

6. **统一 backbone 与训练预算的结果、以及官方最佳配置的结果分开报告。** 前者衡量方法，后者衡量完整系统。  
   **Report backbone-controlled and official-best settings separately.**

7. **至少三个随机种子。** 极低标注预算对样本选择和初始化高度敏感。  
   **Use at least three random seeds.**

8. **报告伪标签覆盖率和质量。** 只报告最终 Dice 无法解释性能来自更多样本还是更准标签。  
   **Report pseudo-label coverage and quality, not only final Dice.**

---

## 11. Related Work 可采用的组织逻辑 / Suggested Related Work Narrative

### 11.1 Conventional Semi-Supervised Medical Image Segmentation

从 Mean Teacher、CPS、Cross Teaching 和 MC-Net+ 说明一致性学习如何通过模型、网络结构或解码器扰动利用无标注数据；再用 BCP、ABD 和 DyCON 说明近期工作开始显式处理标注/未标注分布偏差、扰动可靠性和不确定区域。

Start from Mean Teacher, CPS, Cross Teaching, and MC-Net+ to describe prediction consistency under model or architectural perturbations. Then introduce BCP, ABD, and DyCON as methods that explicitly address empirical distribution mismatch, perturbation reliability, and uncertain regions.

### 11.2 Foundation Models for Semi-Supervised Segmentation

SemiSAM/SemiSAM+ 将 specialist 输出转成 prompt，由冻结 generalist 提供伪监督；SAMatch 将 Match teacher 的预测转成 SAM prompt 并联合微调；CPC-SAM 在双 SAM Decoder 间交叉提示；SC-SAM 与 CPPS-SAM 进一步让 specialist 与 SAM 双向协作。这些方法主要解决**单张图像内部**的伪监督质量和模型协作。

SemiSAM and SemiSAM+ use specialist-generated prompts to query frozen generalists. SAMatch couples Match-based learning with a task-adapted SAM. CPC-SAM performs cross prompting between SAM decoders, while SC-SAM and CPPS-SAM establish bidirectional specialist-generalist cooperation. Their interaction is still predominantly within each individual image.

### 11.3 Cross-Sample Relation and Propagation

ACTION、DACL 和 GraphCL 利用跨样本关系改善表示学习；OP-SAM 与 YoloSeg 则将标注信息从一个 support 向 query 或未标注池传播。但现有方法通常不显式回答：**应该从哪个 Anchor 出发、经过哪些未标注 Bridge、以什么顺序到达 Target。** 这就是 RouteSAM 的切入点。

ACTION, DACL, and GraphCL exploit cross-sample relations for representation learning. OP-SAM and YoloSeg transfer supervision from a labeled support to queries or an unlabeled pool. They do not explicitly determine which labeled Anchor should be used, which unlabeled Bridges should be traversed, or in what order the Target should be reached. This is the gap addressed by RouteSAM.

---

## 12. 参考文献元数据注意事项 / Bibliographic Corrections

当前 `related_work_papers/references.bib` 中有几项建议在写论文前更新：

1. `SemiSAM` 不应只保留 2023 arXiv 信息；正式版本为 BIBM 2024，DOI `10.1109/BIBM62325.2024.10821951`。
2. `SemiSAM+` 已发表于 Medical Image Analysis 106 (2025), 103733，DOI `10.1016/j.media.2025.103733`。
3. `CPC-SAM` 已发表于 MICCAI 2024，DOI `10.1007/978-3-031-72120-5_16`。
4. LaTeX 正文中的 `CPAC-SAM` 很可能是 `CPC-SAM` 的笔误，需要统一名称。
5. `YoloSeg` 已发表于 Medical Image Analysis 112 (2026), 104093，DOI `10.1016/j.media.2026.104093`。
6. `CPPS-SAM` 已发表于 Journal of Visual Communication and Image Representation 119 (2026), 104876，DOI `10.1016/j.jvcir.2026.104876`。

---

## 13. 一句话总结 / One-Sentence Takeaway

> 传统半监督方法通过同图一致性、伪标签、数据混合或特征关系来利用无标注数据；SAM 辅助方法进一步让 foundation model 生成或修正伪监督；RouteSAM 的核心区别是把无标注图像本身组织成从覆盖感知 Anchor 到 Target 的有序 Bridge 路线，并沿路线执行、审核和吸收跨图像监督。

> Conventional SSL learns from unlabeled data through same-image consistency, pseudo labels, data mixing, or representation geometry; SAM-assisted methods additionally use foundation models to generate or refine supervision. RouteSAM differs by organizing unlabeled images into ordered Bridge routes from coverage-aware Anchors to Targets and by executing, auditing, and absorbing supervision along those routes.

