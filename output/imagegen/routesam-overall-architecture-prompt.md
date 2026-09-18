# RouteSAM overall architecture prompt

Use case: scientific-educational

Asset type: publication-ready overall architecture figure for a medical image segmentation research paper

Create a clean, precise, wide 16:9 scientific framework diagram for the paper “RouteSAM: Semantic Routing for Semi-Supervised 2D Medical Image Segmentation via Cross-Image Propagation”. Use a white background, crisp vector-infographic appearance, high resolution, readable typography, restrained pastel colors, thin arrows, generous whitespace, and a polished MICCAI or Medical Image Analysis paper aesthetic. The central contribution must look like propagation-path design, not a generic SAM + U-Net two-branch framework.

Arrange the figure left to right in four regions:

1. “Stage 0  Coverage-Aware Anchor Selection”: Training RGB Images → Frozen SAM3 Image Encoder → Global Descriptor and Local Descriptor → Greedy Coverage → 1% Labeled Anchors. Indicate “training RGB only” and “acquire masks after image selection”.

2. “Stage 1  Semantic Route Generation”: Target-Aware Anchor Retrieval; multiple ordered routes “A → T”, “A → B₁ → T”, and “A → B₁ → ··· → Bₖ → T”; Frozen SAM3 Progressive Propagation with subtext “pseudo-video; GT box prompt on Anchor; no text prompt”; Forward propagation and Return Cycle; Route-Dependent Candidates; Route-Candidate Quality Estimator with “28 label-free features + Ridge”; Round-1 Pseudo Label. Use blue for Anchor, green for Bridges, and orange for Target. Make the multi-route candidate pool the visual focus.

3. “Stage 2  Route-Preserving Specialist Collaboration”: Anchor GT + Quality-Weighted Pseudo Labels → U-Net Specialist → Specialist Audit. Draw a dashed audit-feedback arrow labeled “agreement” toward the route candidates. Accepted Candidates and Anchor Masks → PEFT SAM3 Adaptation → Adapted SAM3 → re-propagation through “Same Frozen Routes”. Include a lock note: “freeze Anchor, Bridges, order, and policy”. Then show Round-2 Candidates → Quality Estimator + Specialist Audit → Round-2 Pseudo Labels → Final U-Net Training.

4. “Deployment”: Single Test Image → Final U-Net → Segmentation Mask. Add badges “Prompt-Free”, “Retrieval-Free”, and “Single-Image Inference”. Separate this region from training with a vertical divider labeled “Inference”.

Add “Training Time Only” below Stages 0–2 and a legend: solid arrow = supervision/data flow; dashed arrow = specialist audit; loop arrow = route-preserving re-propagation.

Use abstract, medically plausible 2D image thumbnails with simple lesion masks. No patient faces, graphic anatomy, logos, watermark, citations, numerical performance claims, fake equations, extra stages, duplicated blocks, dark background, gradients behind text, or Chinese text. Keep all English labels verbatim and correctly spelled, and make arrow directions unambiguous.
