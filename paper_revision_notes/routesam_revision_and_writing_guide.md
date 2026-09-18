# RouteSAM 论文调整建议与写作稿 / Revision and Writing Guide

> **审查对象 / Reviewed file:** `cas-dc-template.tex`  
> **依据 / Evidence base:** 当前实验仓库主线、已收集参考文献及正式论文页面  
> **目标 / Goal:** 在不提前虚构第四章结果的前提下，完成逻辑准确、创新边界清晰的摘要、Introduction、Related Work 和 Method 结构。

---

## 1. 总体判断 / Overall Assessment

当前论文的核心想法是成立的，最值得保留的主线是：

> coverage-aware Anchor selection → ordered Anchor–Bridge–Target routing → progressive SAM3 propagation → route candidate selection → specialist audit → SAM3 adaptation → re-propagation → direct specialist inference.

但当前稿件存在三个主要问题：

1. **创新边界没有与最接近工作正面对齐。** YoloSeg 已经实现“单张标注图像 + SAM2 传播 + 伪标签降噪 + 最终 specialist 推理”；SC-SAM、SemiSAM+ 和 CPPS-SAM 已经覆盖 generalist–specialist 协作。因此不能再把“使用 SAM 生成伪标签”“SAM–U-Net 协作”或“最终单图推理”单独写成主要创新。
2. **论文描述超过当前统一实验协议的完成度。** 自动 Anchor 的跨数据集 Stage 0–1 已完成，但 U-Net 审核、SAM3 适配和重传播目前只在旧 Kvasir 固定 Anchor 协议中做过。摘要和贡献中不能提前写成已被三数据集实验验证的事实。
3. **引用和 Related Work 仍不够准确。** 当前 `.tex` 加载的是模板自带的 `cas-refs.bib`，而真正的论文条目在 `related_work_papers/references.bib`；此外存在错误引用键、缺失引用和方法名称错误。

The paper should therefore position RouteSAM around **propagation-source selection and propagation-path construction**, not around the generic use of SAM or generalist–specialist collaboration.

---

## 2. 必须优先修正的问题 / Priority Corrections

### P0：会导致事实或编译错误 / Correctness and Compilation

| 位置 / Location | 当前问题 / Problem | 建议 / Revision |
|---|---|---|
| Line 174 | 使用 `ravi2025sam`、`carion2026sam`，与收集的 BibTeX key 不一致 | 改为 `ravi2025sam2`、`carion2026sam3` |
| Line 177 | 用 SemiSAM+、SC-SAM、SAMatch 支撑“foundation-model propagation” | 这些是协作/伪标签方法；传播应引用 `zhang2026yoloseg`，直接跨图像转移可引用 `mao2025opsam` |
| Line 244 | `SAM-Match` 名称不标准 | 统一为 `SAMatch` |
| Line 244 | `CPAC-SAM` 很可能是笔误 | 改为 `CPC-SAM`，引用 `miao2024cpcsam` |
| Line 273 | 加载 `cas-refs.bib`，其中仍是模板的社区检测文献 | 改为加载 `related_work_papers/references`，或将相关条目合并进 `cas-refs.bib` |
| Related Work | 几乎没有 `\cite{}` | 每个方法族至少在第一次出现时加入正式引用 |
| Keywords | 目前为空 | 补充 4–6 个关键词 |
| Line 263 | `\section{}\label{}` 为空 | 第三章改为 `\section{Method}` 并给出有效 label |

推荐的 bibliography 调用：

```latex
\bibliographystyle{cas-model2-names}
\bibliography{related_work_papers/references}
```

如果希望保持主文件目录简洁，可以将收集后的条目整理进新的 `references.bib`，再使用：

```latex
\bibliography{references}
```

### P1：会削弱创新性的叙事问题 / Novelty and Positioning

| 当前写法 / Current framing | 风险 / Risk | 应改成 / Better framing |
|---|---|---|
| “现有方法直接把有限监督转移到 Target” | 对 Mean Teacher、SemiSAM+、SC-SAM 等并不准确 | 区分 same-image pseudo supervision、direct support-to-query transfer、ordered route propagation |
| “SAM–U-Net collaboration” 是创新 | SemiSAM+、SC-SAM、CPPS-SAM 已覆盖 | 强调 **propagation-oriented collaboration**：specialist 审核的是 route candidates，并在冻结路线重传播 |
| “最终单图推理” 是创新 | SemiSAM+、YoloSeg 同样做到 | 作为部署属性，不作为首要创新 |
| “Bridge 逐渐缩小语义距离” | 如果没有单调距离约束或统计验证，会被追问 | 改为“selected to provide intermediate semantic states and reduce a difficult one-step transfer” |
| “coverage-aware selection improves robustness” | 当前只有 proxy coverage，缺少 random-anchor 下游传播对照 | 方法部分描述设计目标；结果完成后再写最终性能增益 |
| “adaptation and re-propagation improve pseudo labels” | 旧 Kvasir 实验并未稳定支持 | 在方法中写“produces a second-round pseudo-label set”；最终是否提高由第四章结果决定 |

### P2：结构和表达问题 / Structure and Style

- Introduction 从 SAM 开始，缺少医学标注成本与标准 SSMIS 的铺垫。
- Line 195、198、205–207 多次重复同一 observation 和 method overview。
- Abstract、Highlights、Introduction 和 Contributions 四处几乎逐字重复整条流程。
- Highlights 目前是长段落。Elsevier highlights 通常要求短句，应压缩为 3–5 条简短结论。
- “queried target” 不自然，统一使用 `target image`。
- 建议统一将角色写为 `Anchor`, `Bridge`, `Target`，一般概念则使用小写。
- `generalist–specialist` 中使用的 Unicode dash 可能影响部分 LaTeX 环境，正文建议写成 `generalist--specialist`。

---

## 3. 哪些结论现在能写，哪些要等第四章 / Claim Control

| Claim | 当前证据 / Current evidence | 写作建议 / How to write now |
|---|---|---|
| 自动选择约 1% Anchor | 四数据集 Stage 0 已冻结 | 可以写为已实现方法 |
| 自动选择提高表征覆盖率 | Kvasir、ISIC proxy coverage 已有 | 可以写，但明确是 coverage metric |
| 自动 Anchor 提高最终 Dice | random-anchor propagation 尚未完成 | 暂时不能写 |
| `b0`–`b6` 路线和 SAM3 传播 | 四数据集 Stage 1 已完成 | 可以写为已实现方法 |
| Bridge 路线能优于 direct `b0` | Kvasir、ISIC、BUSI 已有固定深度结果 | 可以写，避免说所有 Target 都提高 |
| 当前 Router 稳定选择最佳路线 | Oracle gap 仍明显 | 写成质量选择模块，并承认排序仍是瓶颈 |
| U-Net 可审核传播候选 | 旧 Kvasir 固定 Anchor 已有原型 | 写成 preliminary feasibility，不与自动 Anchor 主表混用 |
| SAM3 adaptation + re-propagation 稳定提升 | 尚未得到稳定结果 | 不能提前写结果性结论 |
| 自动 Anchor 全流程最终单图推理 | 尚未贯通 | 可写成方法目标，不能写成三数据集已验证事实 |

### 推荐措辞 / Recommended Language

方法描述可以使用：

> The adapted SAM3 is then applied to the frozen semantic routes to produce a second-round pseudo-label set.

在没有结果前不要使用：

> The adapted SAM3 further propagates supervision ... improving pseudo-label quality and final segmentation performance.

实验完成后，只有在三个数据集和多个 seed 上成立时，才改为：

> Re-propagation improves the final specialist by X.X Dice points on average across the three datasets.

---

## 4. 推荐的整篇叙事 / Recommended Paper Narrative

论文应该围绕以下问题逐层推进：

1. 少量标注和大量无标注图像是 SSMIS 的基本场景。
2. 传统方法主要从同一图像的预测一致性、伪标签和数据扰动中获得监督。
3. SAM 辅助方法提高了单张无标注图像的伪标签质量，并建立了 generalist–specialist 协作。
4. YoloSeg/OP-SAM 进一步证明 mask 可以跨图像转移，但没有显式优化“传播路径”。
5. 当 labeled source 和 target 差异很大时，问题不只在于模型是否强，还在于监督从哪里出发、经过什么中间状态。
6. RouteSAM 将无标注池解释为可搜索的 semantic transition space：自动选 Anchor，检索并排序 Bridge，再让 SAM3 沿路线执行传播。
7. specialist 的作用不是重复一个普通协作框架，而是审核 route-generated candidates，并把反馈用于 route-preserving adaptation 和 re-propagation。

一句话创新定义：

> RouteSAM moves semi-supervised segmentation from per-image pseudo-label generation to propagation-path design over the unlabeled training set.

---

## 5. 建议摘要 / Proposed Abstract

下面提供的是**目标完整版摘要**。在第四章实验完成前，最后一句必须保留占位符，不能填写推测结果。

```latex
\begin{abstract}
Learning accurate medical image segmentation models from extremely sparse annotations remains challenging because pseudo supervision generated independently for semantically distant target images is often unreliable. Recent Segment Anything Model (SAM)-assisted methods improve pseudo-label quality through prompt-based refinement or generalist--specialist collaboration, while one-shot propagation methods transfer a labeled mask directly to unlabeled images. However, these approaches generally leave the cross-image propagation path unspecified. We propose RouteSAM, a semantic-routing framework for semi-supervised 2D medical image segmentation under an approximately 1\% annotation budget. RouteSAM first selects coverage-aware labeled Anchors and retrieves semantically related unlabeled Bridge images for each Target. The selected samples are ordered into an Anchor--Bridge--Target route, along which SAM3 progressively propagates segmentation information and produces a set of route-dependent mask candidates. A quality-aware selector identifies reliable candidates for pseudo supervision. To absorb the propagated knowledge into a deployable model, a task-specific U-Net is trained from the route-derived labels and is further used to audit propagation candidates and guide parameter-efficient SAM3 adaptation. The adapted SAM3 re-propagates supervision along the frozen routes to construct a second-round training set, while the final U-Net performs prompt-free single-image inference. Experiments on Kvasir-SEG, BUSI, and ISIC2018 show that RouteSAM achieves [RESULTS], outperforming [STRONGEST COMPARABLE BASELINE] by [DELTA] mean Dice under the same annotation budget. Ablation studies further quantify the effects of Anchor selection, Bridge routing, candidate selection, and generalist--specialist refinement.
\end{abstract}
```

### 摘要中最终必须填入的数字 / Required Final Numbers

- 三个数据集的最终 direct-inference Dice；
- 同标注预算下最强传统 SSL baseline；
- 同标注预算下最强 SAM-assisted baseline；
- 平均增益或每数据集增益；
- 至少一个 route-specific ablation 的可量化结论。

如果第四章最终证明闭环没有稳定增益，应删除摘要中的 adaptation/re-propagation 主贡献，将其降为分析实验，并将主线收缩到 Stage 0–1。

---

## 6. 建议 Highlights 与 Keywords / Proposed Highlights and Keywords

建议 Highlights 保持短句：

```latex
\begin{highlights}
\item RouteSAM organizes unlabeled images into ordered propagation routes.
\item Coverage-aware Anchors improve source diversity under a 1\% label budget.
\item Semantic Bridges replace difficult direct transfer with progressive propagation.
\item A SAM3--U-Net loop audits and re-propagates route-derived supervision.
\item The final specialist supports prompt-free single-image inference.
\end{highlights}
```

如果第四章尚未完成，在内部草稿中可保留第 4–5 条；正式投稿前必须由统一协议结果支撑。

推荐关键词：

```latex
\begin{keywords}
Semi-supervised medical image segmentation \sep
Segment Anything Model \sep
Cross-image propagation \sep
Semantic routing \sep
Pseudo-label learning
\end{keywords}
```

---

## 7. Introduction 推荐写法 / Proposed Introduction

### Paragraph 1：从医学半监督问题开始 / Start from the Medical SSL Problem

```latex
Pixel-level annotation is a major bottleneck in medical image segmentation because accurate delineation requires substantial time and domain expertise. Semi-supervised medical image segmentation (SSMIS) alleviates this burden by learning from a small labeled set together with a substantially larger pool of unlabeled images. Existing methods mainly obtain supervision from pseudo labels or prediction consistency under input, feature, and model perturbations~\cite{tarvainen2017meanteacher,chen2021cps,luo2022crossteaching,wu2022mcnet,bai2023bcp,assefa2025dycon}. Although these strategies make effective use of unlabeled data, the generated supervision is commonly determined for each image independently or through local mixing and regularization, making performance sensitive to confirmation bias and unreliable predictions under extreme label scarcity.
```

### Paragraph 2：引入 SAM，但不要夸大 / Introduce SAM without Overclaiming

```latex
Promptable segmentation foundation models offer a complementary source of prior knowledge. The Segment Anything Model (SAM) family~\cite{kirillov2023segment,ravi2025sam2,carion2026sam3} has demonstrated strong prompt-conditioned segmentation ability after large-scale pretraining. However, direct deployment in medical images remains unreliable because of differences in image formation, target scale, contrast, and morphology~\cite{mazurowski2023samexperiment,ma2024medsam}. Consequently, recent studies have incorporated SAM into label-efficient medical segmentation rather than relying on its zero-shot predictions alone.
```

### Paragraph 3：准确概括 SAM-assisted methods / Characterize SAM-Assisted SSL

```latex
Existing SAM-assisted SSMIS methods primarily improve the supervisory signal assigned to each unlabeled image. SemiSAM and SemiSAM+ convert specialist predictions into prompts and use frozen generalist models to provide additional pseudo supervision~\cite{zhang2023semisam,zhang2025semisamplus}. SAMatch jointly trains a Match-style segmentation model and a task-adapted SAM so that automatically generated prompts yield refined pseudo labels~\cite{xu2024samatch}. CPC-SAM exploits cross prompting between two SAM decoders and enforces invariance to prompt locations~\cite{miao2024cpcsam}, while SC-SAM and CPPS-SAM further introduce bidirectional knowledge transfer between a conventional specialist and SAM~\cite{vu2026scsam,zhang2026cppssam}. These approaches substantially strengthen per-image pseudo supervision, but their interactions are predominantly defined within the same image and do not explicitly determine a cross-image path for supervision transfer.
```

### Paragraph 4：引入跨样本关系 / Discuss Cross-Sample Structure

```latex
Another line of work demonstrates that the unlabeled set contains useful dataset-level structure. ACTION learns global semantic relations and local anatomical contrast~\cite{you2022action}; DACL constructs density-aware neighborhood graphs to compact class features~\cite{tang2024dacl}; and GraphCL models instance relations using graph-based clustering~\cite{wang2024graphcl}. These methods show that cross-sample relations are informative, but they mainly use such relations to regularize representation learning rather than to determine how a segmentation mask should be propagated between images.
```

### Paragraph 5：必须正面对比 OP-SAM 与 YoloSeg / Confront the Closest Propagation Work

```latex
Cross-image mask transfer has also been explored under one-shot supervision. OP-SAM transfers a prior from a labeled support image to a query through feature correlation and iterative prompt evolution~\cite{mao2025opsam}. More recently, YoloSeg uses SAM2 to propagate a single labeled mask to an unlabeled pool and learns a task-specific model from multi-view consensus and divergence regions~\cite{zhang2026yoloseg}. These studies establish the value of foundation-model-driven propagation. However, they do not explicitly optimize an ordered path through selected intermediate unlabeled samples. When the labeled source and target differ substantially in morphology, scale, or appearance, a direct source-to-target transfer may still be difficult even if the resulting pseudo labels are subsequently filtered or regularized.
```

### Paragraph 6：观察与问题定义 / Observation and Research Question

```latex
We observe that an unlabeled medical dataset can be viewed not only as a collection of prediction targets but also as a semantic transition space. Between a labeled source and a difficult target, the pool may contain intermediate images with progressively related lesion appearances. This observation motivates a different question from conventional pseudo-label learning: rather than only asking how to improve the prediction of an unlabeled image, can we design the path through which sparse supervision reaches that image?
```

这里建议避免写“每一步严格减小语义距离”，除非 Method 中给出了单调约束并在实验中验证。更安全的措辞是：

> intermediate images selected to provide smoother local transitions.

### Paragraph 7：方法概述 / Method Overview

```latex
To answer this question, we propose RouteSAM, a semantic-routing framework for semi-supervised 2D medical image segmentation. Under a fixed annotation budget, RouteSAM first selects coverage-aware training samples as labeled Anchors. For each Target, lesion-aware retrieval identifies a relevant Anchor and a set of unlabeled Bridge candidates, which are ordered into an Anchor--Bridge--Target route. SAM3 then treats the route as a pseudo-video and progressively propagates the Anchor mask across the selected images, producing route-dependent target candidates. A quality-aware selector retains reliable candidates for pseudo supervision. A task-specific U-Net absorbs the propagated supervision and audits the route candidates; its domain-specific predictions subsequently guide parameter-efficient SAM3 adaptation. The adapted model is applied to the same frozen routes to construct a second-round pseudo-label set, after which the final specialist performs direct single-image inference without retrieval or prompting.
```

### Paragraph 8：实验概述必须保持可验证 / Experimental Summary

当前阶段建议先写成 TODO 版本：

```latex
We evaluate RouteSAM on Kvasir-SEG, BUSI, and ISIC2018 under an approximately 1\% annotation budget. The experiments compare RouteSAM with conventional SSMIS approaches, SAM-assisted methods, and one-shot propagation baselines. We further isolate the effects of coverage-aware Anchor selection, Bridge depth, route-candidate selection, specialist auditing, SAM3 adaptation, and re-propagation. [Replace this sentence with quantitative findings only after the unified protocol is complete.]
```

不要在结果未完成前写：

> The experimental results demonstrate that ... effectively improves ...

---

## 8. Contributions 推荐写法 / Proposed Contributions

```latex
The main contributions are summarized as follows:
\begin{itemize}
\item We formulate semi-supervised medical image segmentation as a cross-image supervision-routing problem and introduce RouteSAM, which organizes a labeled Anchor, selected unlabeled Bridges, and a Target into an ordered propagation route instead of treating every unlabeled image as an independent pseudo-labeling target.

\item We develop a coverage-aware source-selection and route-construction strategy for extremely sparse annotation. The method selects representative Anchors under an approximately 1\% label budget and retrieves intermediate Bridge images to replace a difficult one-step transfer with a sequence of locally related propagation steps.

\item We introduce a route-aware supervision pipeline in which SAM3 generates multiple route-dependent candidates, a quality-aware module selects pseudo supervision, and a task-specific specialist audits the candidates and guides route-preserving SAM3 adaptation and re-propagation.

\item We establish a unified evaluation protocol on Kvasir-SEG, BUSI, and ISIC2018 that separates source selection, route construction, candidate ranking, collaborative refinement, and final prompt-free inference. [Retain this contribution only after all corresponding experiments are complete.]
\end{itemize}
```

注意：第三条的真正创新必须写成 `route-aware` 或 `route-preserving collaboration`。如果只写 SAM3–U-Net collaboration，会与 SemiSAM+、SC-SAM 和 CPPS-SAM 高度重叠。

---

## 9. Related Work 推荐重写 / Proposed Related Work

### 9.1 Semi-Supervised Medical Image Segmentation

```latex
\subsection{Semi-Supervised Medical Image Segmentation}

Semi-supervised medical image segmentation learns from limited pixel-level annotations and abundant unlabeled images. A dominant family of methods relies on consistency regularization. Mean Teacher uses an exponential-moving-average teacher to provide stable targets under perturbations~\cite{tarvainen2017meanteacher}. Cross Pseudo Supervision trains two independently initialized networks to exchange hard pseudo labels~\cite{chen2021cps}, while Cross Teaching exploits the complementary inductive biases of CNN and Transformer models~\cite{luo2022crossteaching}. MC-Net+ estimates uncertain regions from multiple decoders and imposes mutual consistency with soft pseudo labels~\cite{wu2022mcnet}.

Recent methods improve the reliability of perturbation-based learning and pseudo supervision. BCP reduces the empirical mismatch between labeled and unlabeled data through bidirectional copy--paste~\cite{bai2023bcp}. ABD uses confidence-guided bidirectional patch displacement to control multiple perturbations~\cite{chi2024abd}, whereas DyCON dynamically reweights uncertain voxels and combines consistency with focal contrastive learning~\cite{assefa2025dycon}. Although these methods exploit unlabeled data effectively, their supervisory mechanisms are mainly defined through predictions, perturbations, or local sample mixing. They do not explicitly construct a multi-image path along which sparse segmentation supervision is propagated.
```

### 9.2 Foundation Model-Assisted Label-Efficient Segmentation

```latex
\subsection{Foundation Model-Assisted Label-Efficient Medical Segmentation}

The promptable Segment Anything Model has motivated the use of foundation-model priors in label-efficient medical segmentation. SemiSAM adds a frozen SAM-assisted branch to a conventional consistency framework, using specialist predictions to generate prompts and SAM outputs as additional supervision~\cite{zhang2023semisam}. SemiSAM+ generalizes this design to one or multiple frozen generalists and introduces confidence-aware specialist--generalist collaboration~\cite{zhang2025semisamplus}. SAMatch combines weak-to-strong Match learning with a task-adapted SAM: the Match teacher generates automatic prompts, and SAM returns refined masks for student training~\cite{xu2024samatch}.

Other methods adapt SAM itself using unlabeled data. CPC-SAM employs two SAM decoders that generate prompts and supervision for each other, together with prompt-consistency regularization~\cite{miao2024cpcsam}. SC-SAM uses a U-Net to provide point prompts and pseudo labels for parameter-efficient SAM adaptation while SAM regularizes the U-Net~\cite{vu2026scsam}. CPPS-SAM similarly transfers knowledge bidirectionally between a CNN expert and a SAM generalist through cross prompting and cross pseudo supervision~\cite{zhang2026cppssam}. These methods establish effective generalist--specialist interaction, but the interaction is primarily performed within individual images. RouteSAM is complementary: it focuses on how supervision is organized and propagated across a sequence of related images.
```

### 9.3 Cross-Image Relation Modeling and Propagation

```latex
\subsection{Cross-Image Relation Modeling and Propagation}

Several SSMIS methods exploit relationships among images to improve representation learning. ACTION performs global and local anatomical-aware contrastive distillation~\cite{you2022action}. DACL uses density-aware neighbor graphs to pull sparse features toward high-density class regions~\cite{tang2024dacl}, while GraphCL incorporates instance graphs and graph-based clustering into a teacher--student framework~\cite{wang2024graphcl}. These studies demonstrate that unlabeled medical datasets contain valuable cross-sample structure; however, the learned relations regularize feature geometry rather than define an executable segmentation-propagation path.

Support-conditioned methods perform more direct cross-image transfer. PerSAM and Matcher personalize or prompt SAM from a reference image and mask~\cite{zhang2023persam,liu2023matcher}, while OP-SAM transfers one-shot polyp priors through support--query correlation and iterative prompt evolution~\cite{mao2025opsam}. YoloSeg uses SAM2 to propagate a single annotation to unlabeled images and trains a specialist using multi-view pseudo-label consensus~\cite{zhang2026yoloseg}. In contrast to these direct or support-conditioned approaches, RouteSAM explicitly selects the propagation source and inserts ordered unlabeled Bridges between the Anchor and Target. The resulting route converts the unlabeled pool from a set of passive prediction targets into an active transition space for progressive supervision transfer.
```

### 为什么删掉正文中的两个独立公式 / Why the Two Standalone Equations Should Be Reconsidered

当前 Related Work 用

```latex
A \rightarrow T
```

对比

```latex
A \rightarrow B_1 \rightarrow \cdots \rightarrow T
```

直观但过于简化，容易被审稿人指出 YoloSeg 并非简单的一次 forward。建议将公式保留在 Introduction 的 motivation figure 或 Method problem formulation 中；Related Work 改用文字说明“does not explicitly optimize an ordered intermediate path”。

---

## 10. Motivation Figure 建议 / Figure 1 Revision

当前图片可以保留，但 caption 建议改为：

```latex
\caption{Conceptual comparison between direct and routed cross-image propagation. Given a labeled Anchor $A$ and an unlabeled Target $T$, direct propagation transfers the mask in a single source-to-target step. RouteSAM retrieves and orders unlabeled Bridge images $\{B_i\}_{i=1}^{K}$ and applies SAM3 sequentially along $A\rightarrow B_1\rightarrow\cdots\rightarrow B_K\rightarrow T$. The Bridges are selected to provide intermediate semantic states and reduce the difficulty of a large one-step transfer. The route is constructed only during training; the final specialist performs direct single-image inference. The displayed masks constitute an illustrative example rather than an aggregate comparison.}
```

如果图中显示 direct Dice `0.383`、Bridge Dice `0.934`，必须在 caption 中明确是 illustrative case，并在实验部分补充全 test set 的配对统计，避免被认为只挑选有利样本。

---

## 11. 第三章 Method 推荐结构 / Recommended Method Section

```latex
\section{Method}\label{sec:method}
\subsection{Problem Formulation and Framework Overview}
\subsection{Coverage-Aware Anchor Selection}
\subsection{Anchor and Bridge Retrieval}
\subsection{Ordered Semantic Route Construction}
\subsection{Progressive SAM3 Propagation}
\subsection{Route-Candidate Quality Estimation}
\subsection{Specialist Auditing and SAM3 Adaptation}
\subsection{Route-Preserving Re-Propagation and Final Inference}
\subsection{Training Objectives and Algorithm}
```

### 11.1 Problem Formulation

必须明确：

- labeled budget 是 `K=max(1, round(0.01 N_train))`；
- Anchor 集来自训练集，选中后才开放人工 mask；
- Bridge 和 Target 只能来自未标注训练池；
- validation GT 可用于 Router 拟合，但这意味着“1%”是训练 Anchor 标注预算，而不是整个研究过程只有 1% GT 可见；
- test GT 只能用于最终评估。

### 11.2 Coverage-Aware Anchor Selection

写清楚选择特征、global/local coverage objective、greedy 过程和复杂度。不要在 Method 中写“比 random 更好”，那属于 Experiment。

### 11.3 Anchor and Bridge Retrieval

定义 Target Pooling、support pool、相似度、候选数量和是否排除 Target 自身。必须解释 `lesion-aware` 的具体数学含义，否则改成更中性的 `target-aware` 或 `region-aware`。

### 11.4 Ordered Route Construction

定义 `b0`–`b6`：

- `b0` 是 direct Anchor→Target；
- `bk` 表示 k 个 Bridge，还是第 k 种 route template，必须统一；
- route ordering 是否满足单调相似度、最小总距离或局部平滑目标；
- beam size 32 和 tie-breaking 规则。

### 11.5 Progressive SAM3 Propagation

明确 SAM3 如何把静态图像序列构造成 pseudo-video，第一帧使用 tight GT box，后续帧如何继承 memory/mask，图像如何统一到 256 canvas，以及为什么不使用 text prompt。

### 11.6 Route-Candidate Quality Estimation

这是当前稿件缺失但实验仓库实际存在的重要模块。需要说明：

- 每个 Target 有多少 route candidates；
- raw 与 centered score；
- top-1 与 top-2 的保留/融合方式；
- legacy-28 feature 和 Ridge Router；
- Router 只用 validation GT 拟合；
- Oracle 仅用于分析，不是部署方法。

### 11.7 Specialist Audit and SAM3 Adaptation

需要与 SC-SAM 明确区别：

- SC-SAM 在同一图像上进行 U-Net↔SAM 双向 co-training；
- RouteSAM specialist 审核的是**不同传播路线生成的候选集合**；
- feedback 应影响候选接纳、SAM3 adaptation 或两者；
- 必须冻结第一轮路线，才能把后续变化归因于模型适配，而不是路线重新搜索。

### 11.8 Re-Propagation and Final Inference

写清楚重传播前后唯一改变的是 SAM3 参数还是还包括 Router。推荐最干净的实验是：

> freeze Anchor, Bridge membership, route order, and selection policy; replace only base SAM3 with adapted SAM3.

最终模型是 U-Net 还是 adapted SAM3 必须唯一明确。若论文主张无提示、无需检索、单图推理，最好把 final U-Net 作为唯一 deployment model。

---

## 12. BibTeX 修正建议 / BibTeX Updates

### 添加 ABD

```bibtex
@inproceedings{chi2024abd,
  title     = {Adaptive Bidirectional Displacement for Semi-Supervised Medical Image Segmentation},
  author    = {Chi, Hanyang and Pang, Jian and Zhang, Bingfeng and Liu, Weifeng},
  booktitle = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  pages     = {4070--4080},
  year      = {2024},
  doi       = {10.1109/CVPR52733.2024.00390}
}
```

该 DOI 已通过 CVPR/DBLP 元数据交叉核对。

### 更新 SemiSAM

```bibtex
@inproceedings{zhang2023semisam,
  title     = {{SemiSAM}: Enhancing Semi-Supervised Medical Image Segmentation via {SAM}-Assisted Consistency Regularization},
  author    = {Zhang, Yichi and Yang, Jin and Liu, Yuchen and Cheng, Yuan and Qi, Yuan},
  booktitle = {IEEE International Conference on Bioinformatics and Biomedicine},
  pages     = {3982--3986},
  year      = {2024},
  doi       = {10.1109/BIBM62325.2024.10821951}
}
```

### 更新 SemiSAM+

```bibtex
@article{zhang2025semisamplus,
  title   = {{SemiSAM+}: Rethinking Semi-Supervised Medical Image Segmentation in the Era of Foundation Models},
  author  = {Zhang, Yichi and Lv, Bohao and Xue, Le and Zhang, Wenbo and Liu, Yuchen and Fu, Yu and Cheng, Yuan and Qi, Yuan},
  journal = {Medical Image Analysis},
  volume  = {106},
  pages   = {103733},
  year    = {2025},
  doi     = {10.1016/j.media.2025.103733}
}
```

### 更新 CPC-SAM

```bibtex
@inproceedings{miao2024cpcsam,
  title     = {Cross Prompting Consistency with Segment Anything Model for Semi-Supervised Medical Image Segmentation},
  author    = {Miao, Juzheng and Chen, Cheng and Zhang, Keli and Chuai, Jie and Li, Quanzheng and Heng, Pheng-Ann},
  booktitle = {Medical Image Computing and Computer Assisted Intervention},
  pages     = {167--177},
  year      = {2024},
  doi       = {10.1007/978-3-031-72120-5_16}
}
```

---

## 13. 推荐修改顺序 / Recommended Editing Order

1. 修复 bibliography 文件与 citation keys，使当前稿件能正确解析引用。
2. 重写 Related Work，先把创新边界与最近工作说准确。
3. 重写 Introduction，删除重复段落并正面对比 YoloSeg、OP-SAM、SC-SAM。
4. 根据实验完成状态修改 Abstract、Highlights 和 Contributions。
5. 按 Stage 0 / Stage 1 / Stage 2 写 Method，使论文结构与代码仓库一致。
6. 第四章完成统一闭环后，再回填 Abstract 的最终数字和结果性措辞。

---

## 14. 最关键的写作原则 / Most Important Writing Principle

不要把 RouteSAM 写成“又一个 SAM + U-Net 半监督框架”。更清晰、也更可辩护的定义是：

> RouteSAM is a propagation-path design framework. It determines where sparse supervision starts, which unlabeled images serve as intermediate semantic states, how SAM3 executes the ordered transfer, and how the resulting route candidates are audited and absorbed into a deployable specialist.

这一定义能够同时避开与 SemiSAM+/SC-SAM 的 generalist–specialist 重叠，也能正面对比 YoloSeg 的单源直接传播。
