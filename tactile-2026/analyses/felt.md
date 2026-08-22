# FELT — Generating Tactile Signals from Vision for Visuo-Tactile Manipulation

**arXiv:2607.20683** · USC + Columbia + Starpilot (Z. Li, Ling, Gu, B. Huang, C. Liang, Islam, Bedri, Chirikjian, Y. Li, Nikolaidis, Seita) · Jul 2026 · [site](https://felt-tactile.github.io/)

**One line.** Synthesises **per-finger pressure images from a single RGB frame** at ~20 ms, so vision-only demonstrations can be retro-fitted with tactile channels and policies can run **without a physical tactile sensor at deployment**.

## 1. What "tactile" means here

**Per-finger pressure-array images** from dual-finger grippers with fisheye RGB cameras (UMI-style), predicting **per-finger contact probability and pressure intensity**.

The insight the method rests on: *"when the gripper, object, and contact region are visible, the surrounding visual context carries strong cues about where and how contact occurs, which is enough to predict the resulting pressure distribution."*

## 2. Model — topology-aware decoding

- **Frozen DINOv2 ViT** encoder
- **Lightweight attention-based query decoder** over a **12×32 query grid**, single feed-forward pass
- **Separate left/right branches** with **side embeddings**, plus a **gated panel exchange** between them
- Conv blocks / conv head producing the final pressure maps

The dual-branch design is justified physically: it *"respect[s] the physical topology of dual-finger tactile sensors... capturing the asymmetric contact patterns during interactions such as wiping, insertion, and in-hand rotation."* The gated exchange lets the fingers inform each other without collapsing into a symmetric prediction.

Three deployment modes:
1. **Offline augmentation** of vision-only datasets with generated tactile
2. **Runtime deployment** — generated tactile fed to a policy at ~20 ms latency (RTX 4090), fast enough for closed-loop control
3. **Latent tactile features** — no tactile images at all, no real sensor during policy training *or* deployment

## 3. Results

Four contact-rich tasks — **Tube Insertion, Cup Nesting, Triangle Peg Insertion, Eraser Wiping** — 20 trials each, 60 demonstrations per task, Diffusion Policy.

**Vision + FELT Tactile** improves over vision-only on all final task metrics, **matching real tactile on Cup Nesting and Triangle Peg Insertion** while trailing it on Tube Insertion and Eraser Wiping.

**Per-stage breakdowns** show *where*: grasp success is similar across methods, with the gains concentrated in **reorientation, insertion, nesting and wiping**. On Cup Nesting specifically, *"gripper occlusion makes cup alignment difficult to infer from vision alone, whereas tactile feedback helps the policy adjust contact and retry."*

**The sensor-free result and its honest caveat.** Vision + FELT *Features* — no real tactile at any point — performs **competitively with the real-tactile baseline**, particularly strongly on Triangle Peg Insertion. The proposed explanation is stated as a hypothesis, not a finding:

> *"With only 60 demonstrations per task, the variability and noise in real tactile images may limit their effectiveness as a stable learning signal... Features from G_φ, pretrained on large paired RGB-tactile data, may instead provide a smoother averaged contact representation that is easier to use in this low-data regime. This explanation remains a hypothesis under our current setting and requires further evaluation."*

That is the right way to report an anomalous result, and it is the same denoising explanation that would account for [[hapticvla]] beating its sensored teacher and [[vital]]'s predicted latents outranking ground-truth ones. Three independent observations of the same phenomenon, none yet tested directly.

**Ablations, deployed on the real robot** — the policy is trained with *real* tactile and only the deployment-time tactile input is swapped for FELT variants:

| Variant | Eraser Wiping: Grasp / ≤2 Wipes / ≤3 Wipes / **Final** | Triangle Peg: Grasp / **Insert** |
|---|---|---|
| + Real Tactile | 100 / 55 / 80 / **90%** | 100 / **70%** |
| + FELT Tactile | 100 / 70 / 75 / **75%** | 100 / **70%** |
| − Panel Exchange | 100 / 60 / 75 / **75%** | 100 / **55%** |
| − Dual-Branch Decoder | 95 / 60 / 65 / **70%** | 100 / **55%** |
| − Conv Head | 100 / 65 / 75 / **75%** | 100 / **60%** |

On peg insertion, **panel exchange and the dual-branch decoder each cost 15 points** — the two components encoding finger topology. And the behavioural correlate is reported: the single-decoder variant *"often produces slower, less stable insertion attempts with repeated retries, consistent with its poor tactile prediction quality."*

Note that FELT matches real tactile on peg insertion (70% both) but trails on wiping (75% vs 90%) — sustained sliding contact is harder to predict from a single frame than a discrete alignment event.

## 4. Stated limitations

- Depends on the **variety and quality of paired visuo-tactile data** available for training.
- **Requires visible contact regions**: *"Heavy occlusion can make FELT fail to infer tactile information."* Proposed fixes — extra camera viewpoints, or temporal models preserving information from earlier unoccluded frames. This is a real tension: touch is most valuable precisely when vision is occluded, and that is when FELT degrades.
- Only **pressure-based** sensors; magnetic and gel-based synthesis untried.

## 5. What it adds that the others don't

The **deployment path**. [[egotac]] predicts tactile for human video, [[tacgen]] generates tactile *latents* for representation probes, [[unitac]] generates cross-sensor tactile for classifier adaptation — FELT generates tactile that a **closed-loop policy consumes at 20 ms on a real robot**, and releases the datasets with both real and generated channels. The finger-topology-aware decoder (separate branches + gated exchange), ablated on hardware rather than just on prediction metrics, is a small architectural idea with a measured 15-point payoff.
