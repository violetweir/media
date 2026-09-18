# RouteSAM 论文主线与实验仓库映射（中英双语）

> **论文 / Manuscript:** `RouteSAM: Semantic Routing for Semi-Supervised 2D Medical Image Segmentation via Cross-Image Propagation`  
> **实验仓库 / Experiment repository:** `violet@222.31.141.50:/Data_8TB/lht/PseudoVideo-SAM3-X3-B7-github`  
> **核对版本 / Audited revision:** branch `main`, commit `d4464a91c0115a84e115c24ed0cb9af73dd51f38`  
> **本文档用途 / Purpose:** 将当前 LaTeX 中的论文叙事映射到服务器上的真实实验、结果与完成状态，为前三章写作和第四章实验实现提供统一索引。

---

## 1. 结论摘要 / Executive Summary

### 中文

当前 LaTeX 中的 RouteSAM 由四个连续环节构成：

1. 在固定标注预算下自动选择具有广覆盖性的 **Anchor**；
2. 为每个 Target 构造有序的 **Anchor–Bridge–Target** 语义路线；
3. 使用冻结的 SAM3 沿路线逐步传播，生成多个候选伪标签，并通过质量估计或 Router 选择输出；
4. 用任务专用 U-Net 审核伪标签、指导 SAM3 适配，再沿原路线重新传播，最终得到可直接单图推理的模型。

实验仓库表明，这四个环节目前并不是在同一套协议上完整贯通的，而是由两条实验线组成：

- **主线 A：自动 Anchor 的跨数据集路线传播。** Stage 0（自动 Anchor）和 Stage 1（路线构造、SAM3 传播、Router）已经在 Kvasir-SEG、ISIC2018、BUSI 和 TN3K 上完成并冻结。这是当前最适合支撑论文“语义路线”核心贡献的主实验线。
- **主线 B：Kvasir 固定 Anchor 的 generalist–specialist 闭环。** U-Net 训练、学生审核、B7 重筛、SAM3 LoRA/第二轮筛选等工作只在旧的 Kvasir 固定 8 Anchor 协议中进行过。它可以证明多阶段闭环的可行性，但不能直接当作自动 Anchor 跨数据集协议的最终结果。

因此，前三章可以按照现有概念框架继续写，但第四章实现时必须补齐：**在自动 Anchor 协议下，将 Stage 1 生成的伪标签池真正接入 U-Net 审核、SAM3 适配与重传播，并在 Kvasir、BUSI、ISIC2018 上完成同协议评估。**

### English

The current RouteSAM manuscript describes a four-step pipeline:

1. automatically selecting coverage-aware labeled **Anchors** under a fixed annotation budget;
2. constructing an ordered **Anchor–Bridge–Target** semantic route for each target;
3. progressively propagating supervision with frozen SAM3, producing multiple pseudo-mask candidates and selecting an output through quality estimation or a Router;
4. using a task-specific U-Net to audit pseudo labels and guide SAM3 adaptation, followed by route-based re-propagation and a final model capable of direct single-image inference.

The repository shows that these four steps have not yet been completed under one unified protocol. Instead, the evidence is split into two experiment lines:

- **Line A — cross-dataset routing with automatically selected Anchors.** Stage 0 and Stage 1 are complete and frozen for Kvasir-SEG, ISIC2018, BUSI, and TN3K. This is the strongest experimental basis for the paper's central semantic-routing contribution.
- **Line B — generalist–specialist closure under the older fixed-Anchor Kvasir protocol.** U-Net training, student auditing, B7 re-screening, SAM3 LoRA, and second-round selection were studied only with eight fixed Kvasir Anchors. These experiments establish feasibility, but they are not final results for the newer cross-dataset automatic-Anchor protocol.

The first three chapters can therefore follow the current conceptual narrative. Chapter 4, however, must complete the missing unified experiment: connect the automatically selected Anchor protocol to U-Net auditing, SAM3 adaptation, and re-propagation on Kvasir, BUSI, and ISIC2018.

---

## 2. 论文主线与实验状态 / Manuscript-to-Experiment Status Map

| 论文中的模块 / Manuscript component | 实验实现 / Repository evidence | 数据集 / Datasets | 当前状态 / Status |
|---|---|---|---|
| Coverage-aware Anchor selection | coverage-greedy / global-local facility selection | Kvasir, ISIC2018, BUSI, TN3K | **已完成 / Complete** |
| Lesion-aware Anchor matching | Target Pooling support retrieval and ranking | Kvasir, ISIC2018, BUSI, TN3K | **已完成 / Complete** |
| Ordered Anchor–Bridge–Target route | `b0`–`b6` route family, KNN/patch correspondence, beam search | Kvasir, ISIC2018, BUSI, TN3K | **已完成 / Complete** |
| Progressive SAM3 propagation | frozen SAM3, 256×256 canvas, Anchor GT box prompt, no text prompt | Kvasir, ISIC2018, BUSI, TN3K | **已完成 / Complete** |
| Candidate selection / quality routing | raw or centered scores, top-1/top-2, legacy-28 Ridge Router | Kvasir, ISIC2018, BUSI, TN3K | **已完成 / Complete** |
| U-Net specialist learning | hard/soft pseudo-label training and test evaluation | Kvasir fixed-Anchor only | **原型完成 / Prototype complete** |
| U-Net-based pseudo-label audit | TP candidates plus student re-screening and B7 selection | Kvasir fixed-Anchor only | **原型完成 / Prototype complete** |
| SAM3 domain adaptation | LoRA and second-round selection ablations | Kvasir fixed-Anchor only | **已有实验但未形成稳定统一结论 / Studied, not yet stable** |
| Re-propagation after adaptation | Round-2 pools and selection ablations | Kvasir fixed-Anchor only | **已有实验，尚未证明显著稳定增益 / Implemented, no stable gain yet** |
| Direct single-image inference | trained student / adapted model | Kvasir fixed-Anchor only | **已验证可行 / Feasibility verified** |
| Full RouteSAM under one protocol | automatic Anchor → route → student → adaptation → re-propagation | Kvasir, BUSI, ISIC2018 | **尚未完成 / Open** |

---

## 3. 主线 A：自动 Anchor 的跨数据集语义路线 / Line A: Cross-Dataset Semantic Routing with Automatic Anchors

### 3.1 Stage 0：覆盖感知 Anchor 选择 / Coverage-Aware Anchor Selection

**中文。** 在训练集上先提取全局与局部表征，再通过覆盖式贪心选择确定待标注样本。标注预算固定为训练集大小的约 1%，即

\[
K=\max(1,\operatorname{round}(0.01N_{train})).
\]

Anchor 不只是普通监督样本，也是后续跨图像传播的起点，因此选择目标是让有限起点尽可能覆盖病灶外观、尺度、纹理和表征空间。

**English.** Global and local representations are first extracted from the training set, followed by a coverage-oriented greedy selection procedure. The annotation budget is approximately 1% of the training set. Because Anchors are both labeled examples and propagation sources, the selection objective is to cover diverse lesion appearances, scales, textures, and representation-space regions.

| 数据集 / Dataset | Train / Val / Test | 1% Anchor 数 / Anchors | 协议状态 / Protocol status |
|---|---:|---:|---|
| Kvasir-SEG | 800 / 100 / 100 | 8 | frozen |
| ISIC2018 | 2075 / 259 / 260 | 21 | frozen |
| BUSI | 517 / 64 / 66 | 5 | frozen |
| TN3K | 2303 / 576 / 614 | 23 | frozen |

Kvasir 的代理覆盖率结果为：自动 global-local 选择 `0.852560`，原固定 8 Anchor 为 `0.835373`，随机选择均值为 `0.830501`。ISIC2018 的对应结果为：自动选择 `0.850002`，原固定 21 Anchor 为 `0.829741`，随机选择均值为 `0.827747`。

The proxy coverage score is `0.852560` for automatic global-local selection on Kvasir, compared with `0.835373` for the existing fixed eight Anchors and `0.830501` for the random-selection mean. On ISIC2018, the corresponding values are `0.850002`, `0.829741`, and `0.827747`.

> **重要限制 / Important limitation:** 当前随机 Anchor 对照只完成了覆盖率代理指标，还没有完成相同 Stage 1 下的下游传播对照。因此现在可以写“自动选择提高了表征覆盖率”，但不能写“自动选择已经显著提高最终分割性能”。  
> Only proxy coverage has been compared against random Anchor selection. A controlled downstream propagation experiment is still missing, so improved final segmentation performance must not yet be attributed to the selection method.

**关键目录 / Key directory**

```text
mainline/experiments/automatic_anchor_selection_pilot_20260911/
  policy.json
  coverage_metrics.json
  SELECTIONS_FROZEN.json
  selected_gt_audit.json
  input_provenance.json
  report.md
  COMPLETE.json
```

其他数据集的选择结果随各自 Stage 1 实验一同冻结，尤其包括：

```text
mainline/experiments/isic2018_auto21_tp_validation_20260911/
mainline/experiments/busi_auto5_tp_1pct_20260913/
mainline/experiments/tn3k_busi_factorial_20260915/selection/
```

### 3.2 Stage 1：路线构造与 SAM3 传播 / Route Construction and SAM3 Propagation

**中文。** 对每个 Target，系统先从 Anchor 与未标注池中检索语义相关样本，再生成不同深度的 `b0`–`b6` 路线。`b0` 表示不经过 Bridge 的直接传播，较大的 `b` 表示加入更多中间 Bridge。SAM3 在固定 256×256 画布上执行跨图像传播；Anchor 使用紧致 GT box 作为起始提示，不使用文本提示。每条路线产生一个候选预测，随后通过 raw/centered 质量分数与 top-1/top-2 策略选择或融合候选。

**English.** For each Target, semantically related Anchors and unlabeled samples are retrieved to construct routes of different depths, denoted by `b0`–`b6`. `b0` is direct propagation without a Bridge, while larger values introduce additional intermediate images. Frozen SAM3 performs cross-image propagation on a 256×256 canvas, initialized with a tight ground-truth box on the Anchor and without a text prompt. Each route produces a candidate mask, after which raw or centered quality scores and top-1/top-2 policies select or combine candidates.

Stage 1 的主要可复现实物包括：

```text
mainline/experiments/automatic_anchor_tp_test_20260913/{kvasir,isic2018}/
  policy.json
  FEATURES_FROZEN.json
  SUPPORT_FROZEN.json
  ROUTES_FROZEN.json
  PREDICTIONS_FROZEN.json
  input_sha256.json
  results.json
  report.md
  completion_audit.json

mainline/experiments/automatic_anchor_tp_router_20260913/{kvasir,isic2018}/
  folds_frozen.json
  models_frozen.json
  validation_results.json
  test_choices_frozen.json
  results.json
  completion_audit.json
```

BUSI 与 TN3K 的统一校准实验分别位于：

```text
mainline/experiments/busi_calibration_factorial_20260914/
mainline/experiments/tn3k_busi_factorial_20260915/
```

### 3.3 跨数据集统一结果 / Unified Cross-Dataset Results

以下为冻结的 test mean Dice。四种策略形成一个 2×2 因子设计：是否使用 centered calibration，以及保留 top-1 还是 top-2 候选。

The following table reports frozen test mean Dice under a 2×2 factorial design: raw versus centered calibration and top-1 versus top-2 candidate retention.

| Selection policy | Kvasir | ISIC2018 | BUSI | TN3K |
|---|---:|---:|---:|---:|
| raw top-1 | 0.870848 | 0.868228 | 0.565990 | 0.519585 |
| centered top-1 | 0.860331 | 0.871717 | 0.719992 | 0.556705 |
| raw top-2 | **0.891226** | 0.869276 | 0.656084 | **0.569973** |
| centered top-2 | 0.862761 | **0.875864** | **0.752198** | 0.560661 |
| historical reference | 0.871374 | 0.866573 | 0.566808 | 0.515486 |

主要观察 / Main observations:

- **Kvasir:** raw top-2 最优，说明保留多个候选有帮助，但 centered calibration 反而下降。  
  **Kvasir:** raw top-2 performs best; broader candidate retention helps, whereas centered calibration hurts.
- **ISIC2018:** centered top-2 最优，但增益较小。  
  **ISIC2018:** centered top-2 is best, although the improvement is modest.
- **BUSI:** centered calibration 带来显著提升，centered top-2 达到 `0.752198`。  
  **BUSI:** centered calibration is strongly beneficial, with centered top-2 reaching `0.752198`.
- **TN3K:** raw top-2 最优但总体性能偏低；仓库中的诊断指向病灶尺度差异和候选排序失败。  
  **TN3K:** raw top-2 is best, but absolute performance remains low; repository diagnostics point to lesion-scale mismatch and candidate-ranking failure.

### 3.4 Bridge 的作用 / Effect of Bridge Images

固定每种路线深度并取 rank-1 候选时，直接路线 `b0` 与最佳 Bridge 路线的对比如下：

When route depth is fixed and the rank-1 candidate is evaluated, direct propagation (`b0`) can be compared with the best Bridge route:

| Dataset | Direct `b0` | Best Bridge depth | Best fixed-depth Dice | Oracle over depths |
|---|---:|---:|---:|---:|
| Kvasir | 0.804710 | `b6` | 0.887069 | 0.917615 |
| ISIC2018 | 0.837421 | `b2` | 0.856643 | 0.884316 |
| BUSI | 0.458291 | `b6` | 0.539444 | 0.617141 |

这组结果支持“中间 Bridge 能缓解某些直接 Anchor→Target 传播困难”的核心动机。但是最佳深度随数据集变化，而且 Oracle 明显高于当前 Router，说明主要瓶颈已经从“能否生成好候选”转向“能否可靠识别好候选”。

These results support the core motivation that intermediate Bridges can alleviate difficult direct Anchor-to-Target transfer. However, the best route depth varies by dataset, and the Oracle remains substantially better than the current Router. The main bottleneck is therefore shifting from candidate generation to reliable candidate ranking.

### 3.5 Oracle Gap 与当前瓶颈 / Oracle Gap and Current Bottleneck

| Policy | Kvasir Oracle / gap | ISIC Oracle / gap | BUSI Oracle / gap | TN3K Oracle / gap |
|---|---:|---:|---:|---:|
| raw top-1 | 0.912017 / 0.0412 | 0.883608 / 0.0154 | 0.617141 / 0.0512 | 0.606576 / 0.0870 |
| centered top-1 | 0.906667 / 0.0463 | 0.890596 / 0.0189 | 0.775461 / 0.0555 | 0.615749 / 0.0590 |
| raw top-2 | 0.932897 / 0.0417 | 0.903881 / 0.0346 | 0.776347 / 0.1203 | 0.699971 / 0.1300 |
| centered top-2 | 0.937542 / 0.0748 | 0.911909 / 0.0360 | 0.823649 / 0.0715 | 0.701492 / 0.1408 |

**解释 / Interpretation:** 候选池中通常存在质量更好的 mask，但现有 legacy-28 Ridge Router 尚不能稳定选中它们，尤其是在 BUSI 和 TN3K 上。后续方法改进应优先关注跨数据集可校准的候选质量预测、排序损失和更稳健的保留/融合机制。

Better masks frequently exist in the candidate pool, but the current legacy-28 Ridge Router does not reliably identify them, particularly on BUSI and TN3K. Future method development should prioritize cross-dataset candidate-quality calibration, ranking objectives, and robust candidate retention or fusion.

---

## 4. 主线 B：Kvasir 固定 Anchor 的多阶段闭环 / Line B: Multi-Stage Closure under Fixed Kvasir Anchors

### 4.1 这条线与当前论文的关系 / Relation to the Current Manuscript

**中文。** 当前论文摘要中的“U-Net 审核传播监督 → 指导 SAM3 适配 → 沿同一路线重传播 → 最终单图推理”来自这条旧实验线。它使用 Kvasir 的固定 8 Anchor，而不是 Stage 0 自动选择的 Anchor。因此它应被视为第四章闭环的先导验证和工程资产，不能与主线 A 的跨数据集结果直接拼接成一个端到端实验。

**English.** The manuscript's generalist–specialist narrative—U-Net auditing, SAM3 adaptation, route-based re-propagation, and direct inference—originates from this older experiment line. It uses eight fixed Kvasir Anchors rather than Stage 0's automatically selected Anchors. It should therefore be treated as a pilot validation and reusable implementation asset for Chapter 4, not as an end-to-end continuation of Line A.

### 4.2 早期闭环原型 / Earlier Closure Prototype

仓库方法记录中的早期流程为：

1. 使用 KNN 与 patch correspondence 生成 `b0`–`b6` 路线；
2. SAM3 传播并筛选伪标签；
3. 训练 X3/U-Net specialist；
4. 由学生预测审核候选，形成 B7；
5. 用筛选后的 GT 与伪标签进行 SAM3 LoRA；
6. 用适配后的 SAM3 重传播，再训练最终 specialist。

The earlier pipeline consists of KNN and patch-correspondence route construction, SAM3 propagation, specialist training, student-based candidate auditing, B7 selection, SAM3 LoRA, and route-based re-propagation.

记录中的代表性结果 / Representative recorded results:

- 第一轮高置信池约为 `491 / 792`；结合 Tier A/B 后，X3 使用约 `679` 个伪标签。  
  The first high-confidence pool contains about `491 / 792` images; after adding Tier A/B samples, X3 uses roughly `679` pseudo labels.
- X3 直接 test Dice 约为 `0.8633`。  
  X3 direct test Dice is approximately `0.8633`.
- X3+B7 路线 test Dice 为 `0.899369`，对应 Oracle 为 `0.919805`。  
  X3+B7 route-based test Dice reaches `0.899369`, with an Oracle of `0.919805`.
- 使用 8 GT + 491 伪标签进行 LoRA 后，patch 路线均值为 `0.8662`，Target Pooling 为 `0.8599`。  
  After LoRA with 8 GT and 491 pseudo labels, the mean patch-route result is `0.8662`, while Target Pooling gives `0.8599`.
- 第二轮重训练结果 `0.868266` 未超过对应的 `ft_1pct` 基线 `0.894648`。  
  The second-round retraining result of `0.868266` does not surpass the corresponding `ft_1pct` baseline of `0.894648`.

这说明闭环组件已经存在，但“适配后重传播必然提升”并未被现有实验支持。

This shows that all closure components have been implemented, but the current evidence does not support a claim that post-adaptation re-propagation consistently improves performance.

### 4.3 后续统一学生与重筛实验 / Later Unified-Student and Re-Screening Experiments

#### 单学生 hard/soft 对照 / Single-student hard-versus-soft comparison

`single_student_hard_soft_20260909` 对 588 张训练图进行统一随机采样；两组共享初始权重、每轮顺序和增强随机种子。

| Version | Best epoch | Validation Dice | Test Dice | Test IoU |
|---|---:|---:|---:|---:|
| hard | 436 | 0.819562 | 0.848592 | 0.769222 |
| soft | 556 | **0.827330** | **0.851243** | **0.773038** |

soft-hard 的配对 test 差异均值为 `+0.002651`，95% CI 为 `[-0.013394, 0.017837]`。因此软标签略好，但单 seed 下不能宣称具有稳定显著优势。

The paired mean test difference is `+0.002651`, with a 95% confidence interval of `[-0.013394, 0.017837]`. Soft labels are slightly better, but a stable advantage cannot be claimed from this single-seed experiment.

#### 全量 TP + 学生重筛 / Full TP plus student re-screening

`tp_student_rescreen_20260910` 对 792 张未标注训练图重新审核，不保留旧池特权：

- 伪标签池从 580 增加到 620；保留 580，新增 40；
- validation-best：Dice `0.826621`，test Dice `0.858330`；
- 固定 816 epoch 终点：test Dice `0.869478`。

The pseudo-label pool increases from 580 to 620 samples, retaining all previous samples and adding 40. The validation-selected checkpoint reaches `0.858330` test Dice, while the pre-specified final checkpoint reaches `0.869478`. These values must be reported separately because test performance must not be used to select the checkpoint.

#### B7 Round-2 Pool A / B7 second-round Pool A

`round2_pool_a_20260910` 在冻结第一轮学生后重新评估全部 792 张图，并依据 validation 选择 B7 阈值 `0.97`：

- Pool A：`428 / 792`，覆盖率 `54.04%`；
- 事后训练 GT 审计的伪标签平均 Dice：`0.934062`；
- 与上一轮 620 张池相比：保留 423、移除 197、新增 5；
- 该平均 Dice 是冻结池后的审计结果，不是独立 test 性能，也没有用于逐图删样本。

After freezing the first-round student, all 792 images are re-evaluated. A validation-calibrated B7 threshold of `0.97` retains `428 / 792` images. The post-freeze audit reports a mean pseudo-label Dice of `0.934062`; this is an audit against hidden training masks, not an independent test result and not a per-sample selection signal.

#### 448 张 TP-only 学生补测 / Evaluation of earlier 448-sample TP-only students

| Student | Checkpoint | Validation Dice | Test Dice |
|---|---|---:|---:|
| S2 | best | 0.816123 | 0.846007 |
| S2 | final | 0.810172 | 0.838730 |
| S3 | best | 0.814231 | **0.860283** |
| S3 | final | 0.806084 | 0.853202 |

该对照进一步说明：不同伪标签池、采样、损失权重和模型选择规则会同时影响结果，不能把变化简单归因于某一个模块。

This comparison further demonstrates that pseudo-label membership, sampling, loss weighting, and checkpoint selection all affect performance. Differences cannot be attributed to a single module without a controlled ablation.

### 4.4 关键实验目录 / Key Experiment Directories

```text
mainline/experiments/single_student_hard_soft_20260909/
  config.json
  data/summary.json
  results.json
  report.md
  completion_audit.json
  final_checkpoint_test/

mainline/experiments/tp_student_rescreen_20260910/
  policy.json
  teacher.json
  TRAIN_DECISIONS_FROZEN.json
  data/pool_summary.json
  results.json
  report.md
  b7_new_student/

mainline/experiments/round2_pool_a_20260910/
  policy.json
  THRESHOLD_FROZEN.json
  ALL_SELECTIONS_FROZEN.json
  POOL_A_FROZEN.json
  comparison_580.json
  results.json
  report.md

mainline/experiments/round2_selection_ablation_20260910/
  FROZEN_CALIBRATION.json
  POOLS_FROZEN.json
  screening_results.json
  training_data_audit.json
  A0.yaml
  A1.yaml

mainline/experiments/tp448_s2_s3_test_20260912/
mainline/experiments/tp448_single_student_soft_20260912/
mainline/experiments/tp_tracker_endpoint_20260910/
```

---

## 5. 按数据集汇总的完成状态 / Dataset-Wise Completion Matrix

| Dataset | 自动 Anchor / Auto Anchor | 路线传播 / Routing | Router / calibration | U-Net audit | SAM3 adaptation | Re-propagation | Direct inference |
|---|---|---|---|---|---|---|---|
| Kvasir | complete | complete | complete | fixed-Anchor prototype | fixed-Anchor prototype | fixed-Anchor prototype | fixed-Anchor prototype |
| ISIC2018 | complete | complete | complete | open | open | open | open |
| BUSI | complete | complete | complete | open | open | open | open |
| TN3K | complete | complete | complete + diagnostics | open | open | open | open |

> **写作边界 / Writing boundary:** Kvasir 的 closure 结果与自动 Anchor 的 Kvasir Stage 0–1 结果来自不同实验协议。论文表格和消融必须明确分组，除非第四章重新跑通统一协议。  
> Kvasir closure results and automatic-Anchor Stage 0–1 results come from different protocols. They must remain separated in paper tables and ablations until Chapter 4 reruns the full unified pipeline.

---

## 6. 当前可以安全写入论文的结论 / Claims Supported by Current Evidence

### 可以写 / Supported now

1. **极低标注预算下的覆盖感知 Anchor 选择已经实现且可复现。** 选择清单、策略、输入来源和覆盖指标均已冻结。  
   **Coverage-aware Anchor selection is implemented and reproducible under an approximately 1% annotation budget.**

2. **Anchor–Bridge–Target 路线构造与 SAM3 逐步传播已在多个数据集上完成。**  
   **Anchor–Bridge–Target route construction and progressive SAM3 propagation have been completed across multiple datasets.**

3. **Bridge 路线在 Kvasir、ISIC2018 和 BUSI 上均可优于直接 `b0` 传播，但最佳深度具有数据集依赖性。**  
   **Bridge routes can outperform direct `b0` propagation on Kvasir, ISIC2018, and BUSI, although the optimal depth is dataset-dependent.**

4. **候选生成具有较大潜力，当前限制主要来自候选质量排序。** Oracle gap 为这一点提供了直接证据。  
   **Candidate generation has substantial headroom, while candidate-quality ranking is a major bottleneck, as shown by the Oracle gap.**

5. **在旧 Kvasir 固定 Anchor 协议中，U-Net 审核、伪标签重筛与直接单图推理已经验证可行。**  
   **Under the older fixed-Anchor Kvasir protocol, U-Net auditing, pseudo-label re-screening, and direct single-image inference have been shown to be feasible.**

### 暂时不能写成最终结论 / Not yet supported as final claims

1. 自动 Anchor、跨图像路线、多轮闭环已经在三个论文数据集上端到端完成；  
   the complete automatic-Anchor-to-multi-round pipeline has been validated end-to-end on all three manuscript datasets;
2. 自动 Anchor 比随机 Anchor 在最终 Dice 上稳定更优；  
   automatic Anchor selection consistently improves final Dice over random Anchor selection;
3. U-Net 指导的 SAM3 适配与重传播必然提高伪标签或最终分割性能；  
   U-Net-guided SAM3 adaptation and re-propagation consistently improve pseudo labels or final segmentation;
4. 当前跨数据集主线已经实现无需检索与路线构造的直接测试推理；  
   the current cross-dataset mainline already supports direct test-time inference without retrieval or routing;
5. 当前所有 test 结果都来自完全盲测。Router 使用 validation GT 拟合，且 test 已在研究过程中多次查看，因此论文中需要透明说明。  
   all current test results constitute a fully blind evaluation. The Router is fitted with validation labels, and the test sets have been inspected repeatedly during development; this must be disclosed transparently.

---

## 7. 第四章需要实现的统一闭环 / Unified Closure Required for Chapter 4

### 建议的正式实验顺序 / Recommended Formal Experiment Sequence

1. **冻结共同协议。** 固定数据划分、约 1% 标注预算、自动 Anchor、特征提取器、路线搜索空间、SAM3 权重和 Router 训练规则。  
   **Freeze the shared protocol:** splits, annotation budget, automatic Anchors, feature extractor, route search space, SAM3 weights, and Router fitting rules.

2. **生成第一轮伪标签池。** 对 Kvasir、BUSI、ISIC2018 使用同一 Stage 1 逻辑导出候选、选择结果、置信度和完整哈希。  
   **Generate the first-round pseudo-label pool** for all three datasets with the same Stage 1 logic and frozen artifacts.

3. **训练统一 specialist。** 使用 Anchor GT 与第一轮伪标签训练 U-Net，并只按 validation 选择 checkpoint。  
   **Train a unified specialist** from Anchor GT and first-round pseudo labels, selecting checkpoints only on validation data.

4. **进行学生审核。** 用 specialist 预测与 SAM3 多路线候选的一致性形成可解释的接纳/拒绝/重排规则。  
   **Perform student auditing** using agreement between specialist predictions and SAM3 route candidates.

5. **适配 SAM3。** 比较仅 Anchor GT、Anchor GT + 高置信伪标签，以及是否使用学生软目标。  
   **Adapt SAM3** using controlled variants: Anchor GT only, Anchor GT plus high-confidence pseudo labels, and optional student soft targets.

6. **沿冻结路线重传播。** 路线成员与顺序保持不变，只替换适配后的 SAM3，以隔离 adaptation 的作用。  
   **Re-propagate along frozen routes,** changing only the adapted SAM3 so that the effect of adaptation is identifiable.

7. **训练最终 specialist 并直接测试。** 测试时不检索 Anchor、不构造路线，报告单图推理性能与开销。  
   **Train the final specialist and evaluate direct single-image inference** without Anchor retrieval or route construction.

8. **做跨 seed 与配对统计。** 至少三个 seed；报告 mean±std、逐样本配对差异和置信区间。  
   **Run multiple seeds and paired statistics:** at least three seeds, mean±std, per-image paired differences, and confidence intervals.

### 必需消融 / Required Ablations

| Ablation | 目的 / Purpose |
|---|---|
| random Anchor vs coverage-aware Anchor | 验证 Stage 0 的下游价值 / establish downstream value of Stage 0 |
| `b0` direct vs Bridge routes | 验证语义桥接 / validate semantic bridging |
| single best route vs top-2 retention/fusion | 验证候选宽度 / evaluate candidate breadth |
| no Router vs current Router vs improved ranker | 隔离候选选择瓶颈 / isolate candidate-ranking bottleneck |
| no student audit vs student audit | 验证 U-Net 审核 / validate specialist auditing |
| base SAM3 vs adapted SAM3 | 验证域适配 / validate domain adaptation |
| no re-propagation vs re-propagation | 验证第二轮闭环 / validate the second round |
| route inference vs final direct specialist | 比较训练期机制与部署模型 / compare training mechanism with deployment model |

### 每个正式实验应保存的内容 / Required Artifacts per Formal Experiment

```text
protocol.json or policy.json
input_hashes.json
SELECTIONS_FROZEN.json
ROUTES_FROZEN.json
PREDICTIONS_FROZEN.json
models_frozen.json
validation_results.json
test_choices_frozen.json
results.json
report.md
completion_audit.json
```

---

## 8. 建议的论文实验表组织 / Suggested Paper Table Organization

### Table 1 — Experimental protocol

列出数据集划分、1% Anchor 数、图像尺寸、SAM3 提示、U-Net 结构、Router 输入维度和 checkpoint 选择规则。

Report dataset splits, Anchor counts, image resolution, SAM3 prompts, U-Net architecture, Router feature dimensionality, and checkpoint-selection rules.

### Table 2 — Stage 1 main comparison

以 Kvasir、BUSI、ISIC2018 为主列，比较 direct `b0`、最佳固定 Bridge 深度、部署策略和 Oracle。TN3K 可放补充材料或 failure analysis。

Use Kvasir, BUSI, and ISIC2018 as the main columns; compare direct `b0`, the best fixed Bridge depth, the deployable policy, and the Oracle. TN3K can be placed in supplementary failure analysis.

### Table 3 — Factorial calibration study

报告 raw/centered × top-1/top-2 的统一四格结果，突出校准不是对所有数据集都同向有效。

Report the full raw/centered × top-1/top-2 factorial study, emphasizing that calibration does not affect every dataset in the same direction.

### Table 4 — Full closed-loop ablation

仅在自动 Anchor 协议完成后加入：Stage 1 pseudo labels → U-Net audit → SAM3 adaptation → re-propagation → final direct inference。

Add this table only after completing the automatic-Anchor protocol end-to-end: Stage 1 pseudo labels → U-Net audit → SAM3 adaptation → re-propagation → final direct inference.

---

## 9. 服务器文档索引 / Server Documentation Index

以下文件应作为后续写作与实现的优先事实来源：

The following files should be treated as the primary sources of truth for subsequent writing and implementation:

```text
docs/stage0_stage1.md
  当前自动 Anchor + Stage 1 的冻结定义、结果、Oracle gap 和开放问题
  Frozen definition, results, Oracle gaps, and open issues for automatic Anchors and Stage 1

docs/cross_dataset_1pct.md
  跨数据集 1% 标注预算的主结果与校准解释
  Main cross-dataset 1% results and calibration analysis

docs/method_en.md
docs/method_cn_v2.md
  旧 Kvasir 固定 Anchor 多阶段闭环的方法记录
  Earlier fixed-Anchor Kvasir multi-stage method descriptions

docs/kvasir_versions.md
docs/kvasir_program.md
  Kvasir 各版本关系、实验演化与注意事项
  Kvasir version relationships, experiment evolution, and caveats

mainline/reproduction_guides/automatic_selection_20260914/
  自动选择协议的复现包
  Reproduction bundle for automatic selection
```

---

## 10. 最终主线定义 / Final Working Definition of the Mainline

### 中文

接下来论文与代码应统一使用下面这条主线定义：

> RouteSAM 在约 1% 标注预算下自动选择覆盖训练分布的 Anchor；针对每个未标注 Target，从训练池中检索并排序语义相关的 Bridge，构造 Anchor–Bridge–Target 路线；冻结或适配后的 SAM3 沿路线逐步传播，形成多候选伪监督；任务专用 U-Net 学习并审核这些伪标签，其反馈用于筛选训练数据和适配 SAM3；适配后的 SAM3 沿冻结路线重传播，最终训练出无需路线检索的单图分割模型。

其中，自动 Anchor、路线构造和第一轮传播已经有跨数据集实验支撑；U-Net 审核、SAM3 适配、重传播和最终直接推理需要在同一个自动 Anchor 协议中重新实现和验证。

### English

The manuscript and codebase should use the following unified definition going forward:

> Under an approximately 1% annotation budget, RouteSAM automatically selects Anchors that cover the training distribution. For each unlabeled Target, it retrieves and orders semantically related Bridge images to construct an Anchor–Bridge–Target route. Frozen or adapted SAM3 progressively propagates supervision along the route and generates multiple pseudo-mask candidates. A task-specific U-Net learns from and audits these pseudo labels, providing feedback for data screening and SAM3 adaptation. The adapted SAM3 then re-propagates supervision along the frozen routes, and the refined pseudo labels train a final segmentation model that performs direct single-image inference without route retrieval.

Automatic Anchor selection, route construction, and first-round propagation are currently supported by cross-dataset experiments. U-Net auditing, SAM3 adaptation, re-propagation, and final direct inference still need to be implemented and validated under the same automatic-Anchor protocol.

