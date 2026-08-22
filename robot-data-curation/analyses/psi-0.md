# Ψ₀ (Psi-0) — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Ψ₀: An Open Foundation Model Towards Universal Humanoid Loco-Manipulation |
| **Org** | Physical Superintelligence Lab (RSS 2026; Best Paper, 2nd 3D-LLM/VLA Workshop @ CVPR 2026) |
| **Date** | 2026-03 |
| **Artifact** | arXiv 2603.12263 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — full paper + promised open-source data-processing pipeline** |
| **Corpus** | **~800 h** human video (EgoDex, ~900M frames) + **~30 h** real humanoid robot data (Humanoid Everyday, ~3M frames) |
| **Stance** | **The counter-example in this survey: curation by *exclusion*, not aggregation.** |

## 1. The contrarian thesis

Ψ₀ is the most useful negative control in the whole set. Where Qwen-RobotManip, ABot-M0, and GR00T pool as many heterogeneous sources as their filters will admit, Ψ₀ argues the opposite:

> *"in contrast to approaches that scale with noisy Internet clips or heterogeneous cross-embodiment robot datasets, we demonstrate that pre-training on high-quality egocentric human manipulation data followed by post-training on domain-specific real-world humanoid trajectories yields superior performance."*

Reported outcome: **~800 h human + ~30 h robot beats baselines pretrained on >10× as much data by >40% absolute overall success rate.** The "cleaning pipeline" here is largely the decision about which corpora never enter.

## 2. Sources & scale

| Stage | Dataset | Scale | Role |
|---|---|---|---|
| Pre-training | **EgoDex** | ~900M frames (~800 h), per-frame global transforms for 7 spine joints, 2 arms, 21 joints/hand | Autoregressive next-action-token prediction in *task space* |
| Post-training | **Humanoid Everyday** | ~3M frames real teleoperated | Flow-based action expert in *joint space* |
| Fine-tuning | Own teleop | **80 trajectories/task** | Long-horizon dexterous loco-manipulation |

Single-source pretraining is a deliberate choice: EgoDex is chosen because it is *already* clean (multi-camera on-device VIO tracking), sidestepping the cross-dataset convention chaos that forces five-stage filters elsewhere.

## 3. Ingestion & normalization

- **Coordinate re-anchoring** — all actions transformed into the **current head-camera frame**. This is the same camera-centric trick Qwen-RobotManip uses, applied to a single source: it makes actions egocentrically consistent and directly transferable to a head-mounted robot camera.
- **Frame-rate resampling** — upsampled ×3 for pretraining efficiency; separately, action data is **down-sampled 30 Hz → 10 Hz** for FAST tokenization.
- **Outlier-robust normalization** — *"Due to the presence of extreme outliers in EgoDex, action values are normalized using the 1st and 99th quantiles."* Quantile clipping is used **in place of** an explicit outlier-removal stage: the noise is bounded rather than excised.
- **State inputs omitted entirely during pretraining** — a deliberate reduction of the input surface to avoid the state-stream corruption that Stage-2-style filters elsewhere exist to catch.

## 4. Filtering & quality control

Minimal by design. There is no multi-stage rejection cascade because the corpus is pre-selected for cleanliness. The quality controls that do exist:
- q1/q99 quantile clipping (above).
- **Retrained FAST tokenizer** — the released FAST tokenizer was found unsuitable, so it is retrained from scratch on **500,000 randomly sampled actions** (horizon 1, vocab 2048, scale 100). Tokenizer mismatch is treated as a data-representation defect.
- Teleoperation-side filtering: for the locomotion channel, **clipping and filtering suppress noise from natural human body sway** before commands are recorded.

## 5. Action-space harmonization / retargeting

- **48-DoF task-space representation** via the H-RDT data-processing script, with matching dataset statistics.
- **Embodiment padding, not merging** — Humanoid Everyday spans G1+Dex3-1 and H1+Inspire Hand with different finger counts; these are padded to 36-DoF and 32-DoF respectively, with padded dims corresponding to lower-body signals absent from the source. Padding is explicit and masked rather than implicit zero-fill.
- **Teleop retargeting** — MANUS gloves give finger tracking; thumb/index/middle retargeted to the 3-finger Dex3-1. VR headset + wrist trackers → upper-body IK; waist/foot trackers → high-level locomotion commands.
- **Rejected approach, stated explicitly**: they do **not** retarget whole-body SMPL motion end-to-end (unlike TWIST2, SONIC), because it produces *"foot drifting, unstable lower body motion, and excessive small corrective steps that hinder policy learning."* Instead the lower body is reduced to (vx, vy, vyaw, pyaw) commands driven by an RL policy. This is a data-quality decision expressed as an architecture decision.

## 6. Augmentation
**Random action-token masking** during training: mask the first `d ∈ [1, d_max=6]` action tokens and exclude them from the loss, training the model to predict conditioned on preceding clean context. Functions as noise-robustness regularization at the token level.

## 7. Mixture weighting
Sequential, not blended: **200k steps on EgoDex only, then 30k steps on Humanoid Everyday only** (230k total, ~10 days). Curriculum by stage rather than a sampled mixture — which avoids having to solve the mixture-weight problem at all.

## 8. Scaling evidence

Ψ₀ runs a **data-ablation in the shrinking direction**, which is rarer and more informative than a scaling curve:
- Pretraining on **only 10% of EgoDex**, with post-training and fine-tuning protocols held fixed, *"leads to significantly worse performance on certain tasks and inferior overall performance."*
- So data volume matters — but the headline result is that **800 h of *the right* data beats >8,000 h of mixed data.** Ψ₀ therefore claims a *data-quality* scaling axis rather than a raw-volume one.
- Downstream efficiency: new long-horizon dexterous loco-manipulation skills from **as few as 80 trajectories**.

## 9. What they do not do
- No cross-embodiment robot data pooling — explicitly rejected.
- No Internet video — explicitly rejected as "noisy Internet clips".
- No synthetic/simulation augmentation of the pretraining corpus.
- No VLM co-training stream to preserve semantics (contrast Qwen-RobotManip's 28M VL mixture).
- Consequently: no dedup, no VLM instruction-consistency audit, no video-quality filter — because the ingest is a single curated academic dataset.

## 10. Transferable takeaways
1. **Source selection is the cheapest filter.** A pre-cleaned single source removes the need for most of the cascade other works build.
2. **Quantile clipping as outlier policy** — bound the noise instead of hunting it, when the source is otherwise trustworthy.
3. **Retarget conservatively.** Ψ₀'s finding that end-to-end whole-body SMPL retargeting injects unusable noise is a concrete warning for every human-video pipeline that assumes retargeting is lossless.
4. **Ablate downward.** A "10% of the corpus" run is a cheap way to establish that data volume is actually binding.
5. Read Ψ₀ against **EgoScale** and **DYNA-2**, which take the opposite bet (20,854 h and 1,000,000 h of human video respectively) and report log-linear returns. The disagreement about whether volume or purity dominates is the single sharpest open question in this survey.
