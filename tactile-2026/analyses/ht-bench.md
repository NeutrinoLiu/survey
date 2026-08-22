# HT-Bench — Benchmarking and Learning Dexterous Full-Hand Tactile Representations with Egocentric Vision

**arXiv:2606.19161** (v2, Aug 2026) · Beihang · Rimbot · BUPT · ShanghaiTech · Tsinghua · CAS · Bigai (Huang, Wu, Jiang, … Jiao, Zhong) · Jun 2026

**One line.** Gives up on a universal tactile benchmark and instead bets on one scalable pairing — egocentric vision + full-hand touch — then builds four evaluation tracks that test *representations* rather than policies.

## 1. What "tactile" means here

**Full-hand pressure distribution**, represented as a normalised single-channel map `t ∈ [0,1]^{1×224×224}` — i.e. touch is handled as an image, not a taxel vector. Paired with **egocentric RGB**, which supplies hand-centric interaction context.

The framing is explicitly a retreat from universality. The authors argue a universal tactile benchmark is impractical because sensors differ in hardware principle, spatial layout, signal modality and mounting, while embodiments differ in end-effector morphology and contact pattern. So rather than normalise across all of that, they pick one regime they believe *scales* — egocentric vision + distributed full-hand contact — and standardise inside it.

## 2. Data curriculum

| | |
|---|---|
| RGB frames | **~10 M** |
| Tactile frames | **~7.8 M** |
| Tasks | **226** |
| Scenes | home, electronics workbench, chemistry lab, retail, workbench, outdoor, other |

Built by extending open-source tactile/visuo-tactile datasets (OpenTouch, TouchAnything) with newly collected real full-hand tactile sequences.

**Split discipline is the notable part**: a **task-level OOD split** — one interaction task is held out entirely, and the remaining tasks split 9:1 into train and in-distribution test. Generalisation is therefore measured across tasks, not across random frames. Of the benchmarks compared (Sparsh, AnyTouch 2, OpenTouch, TouchAnything), only TouchAnything and HT-Bench have an OOD split at all.

## 3. Model — HandTouch

A **vector-quantised vision–tactile encoder**, trained in three progressive stages that map one-to-one onto the benchmark's evaluation axes:

- **Stage 1 — spatial topology.** Convolutional patch projection → learnable positional embeddings → **8-layer ViT encoder** → continuous latents `Z_e ∈ R^{N×D}` → **factorised vector quantiser** with a shared codebook of **K = 2048** entries at a bottleneck dimension d ≪ D. Quantisation happens in the low-dimensional projected space (`W_in ∈ R^{d×D}`, nearest codebook entry, then `W_out ∈ R^{D×d}` back to encoder space).
- **Stage 2 — cross-modal alignment.** Masked tactile inpainting conditioned on egocentric vision.
- **Stage 3 — temporal dynamics.** Multimodal tactile frame prediction.

Plus a **drop-based training strategy** (tactile corruption regularisation).

## 4. How tactile enters the model

Tactile is tokenised into non-overlapping patches and pushed through a **discrete token space shared with the visual conditioning path**. The design bet is that a discrete bottleneck forces the encoder to represent contact *structure* rather than fit pixel statistics — and the ablation supports exactly that (below).

## 5. Experiment setup

Four tracks, each probing a different property:

1. **Fine-grained tactile similarity retrieval** — 1-vs-20; candidates ranked by SSIM give the structural reference ordering, compared against cosine-similarity ordering from learned embeddings. Metrics Hit@1, Recall@5.
2. **Masked tactile inpainting** — mask regions, reconstruct from remaining tactile + visual cues. Simulates sensor failure/degraded taxels. Reported over full map (F-) and masked holes (H-).
3. **Vision-to-tactile synthesis (RGB→Tac)** — predict the pressure distribution from a single RGB frame.
4. **Multimodal tactile frame prediction** — predict t_T from v_{T−2} and t_{T−2}. Reported separately because the baselines are single-frame encoders.

Metric: **contact IoU**, cIoU = Σ min(P, P̂) / Σ max(P, P̂) over pixel pressures — a soft-IoU that respects magnitude, not just a binary contact mask.

Downstream: four real contact-rich tasks (board cleaning, pear picking, water pouring, sand shovelling), 15 trials each.

## 6. Does tactile actually help?

**Benchmark results** (best baseline vs HandTouch):

| Track | Metric | Best baseline | HandTouch |
|---|---|---|---|
| Retrieval | Hit@1 ↑ | 94.27 (ViT) | **99.27** |
| Retrieval | Rec@5 ↑ | 74.65 (ViT) | **85.23** |
| Inpainting (test) | F-RMSE ↓ | 0.022 (ViT) | **0.009** |
| Inpainting (test) | F-cIoU ↑ | 0.762 (ViT) | **0.912** |
| Inpainting (OOD) | F-cIoU ↑ | 0.727 (ResNet-18) | **0.800** |
| RGB→Tac (test) | F-cIoU ↑ | 0.642 (CNN) | **0.689** |
| RGB→Tac (OOD) | F-cIoU ↑ | 0.447 (CNN) | **0.457** |

Two honest asymmetries the paper flags itself: **VQ-VAE is the worst model on retrieval** (Hit@1 63.60), i.e. reconstruction-oriented discrete latents alone lose the structural cues fine-grained matching needs — the discrete bottleneck only helps in combination with the other objectives. And under OOD, **ResNet-18 beats HandTouch on hole-region metrics** (H-cIoU 0.620 vs 0.581): global tactile-map generalisation is good, precise reconstruction of severely corrupted local regions on unseen tasks is not solved.

Note also how small the OOD RGB→Tac margin is — 0.457 vs 0.447 cIoU, and every method sits near 0.45. Vision-to-tactile synthesis on unseen tasks is essentially unsolved by all five encoders.

**Ablations** (cIoU):

| Variant | Inpaint H-cIoU test | Inpaint H-cIoU OOD | RGB→Tac F-cIoU test | RGB→Tac F-cIoU OOD |
|---|---|---|---|---|
| w/o drop | 0.712 | 0.567 | 0.678 | 0.453 |
| w/o codebook | **0.765** | 0.529 | 0.708 | 0.454 |
| w/o vision | 0.698 | 0.556 | 0.467 | 0.415 |
| Full | 0.761 | **0.581** | 0.689 | **0.457** |

The codebook row is the clean result: removing it **improves in-distribution** inpainting (0.765 vs 0.761) while cutting OOD H-cIoU from 0.581 to 0.529. The discrete bottleneck buys transferability at a small in-distribution cost — exactly what a bottleneck should do, and rarely demonstrated this cleanly. Removing vision collapses RGB→Tac (0.689 → 0.467), which is unsurprising, but it also hurts *inpainting*, showing egocentric context contributes to tactile reconstruction proper.

**Real-world downstream** (success %, 15 trials each):

| Method | Board clean | Pear pick | Water pour | Sand shovel | Avg |
|---|---|---|---|---|---|
| ResNet-Scratch | 20.0 | 53.3 | 13.3 | 20.0 | 26.7 |
| ResNet-Trained | 46.7 | 73.3 | 33.3 | 46.7 | 50.0 |
| ViT | 40.0 | 73.3 | 33.3 | 33.3 | 45.0 |
| **HandTouch** | **66.7** | **86.7** | **53.3** | **66.7** | **68.3** |

+18.3 points over the best baseline. Note water pouring is hardest for everyone (53.3% at best) — precise contact *and* motion coordination is the regime where a good tactile encoder still isn't enough.

**Stated limitation.** Egocentric + full-hand only: no fingertip optical sensors, no F/T, no skin-like taxel arrays, no non-hand embodiments. So the "scalable regime" bet is untested outside its own regime.

## 7. What it adds that the others don't

A benchmark that evaluates **encoders, not policies**, on four properties that can be measured without a robot — and a task-level OOD split that makes the generalisation claim falsifiable. The codebook ablation is the cleanest published evidence that discrete tactile bottlenecks trade in-distribution fit for transfer. Contrast [[tacverse]], which measures cross-*sensor* shift; HT-Bench measures cross-*task* shift with the sensor held fixed.
