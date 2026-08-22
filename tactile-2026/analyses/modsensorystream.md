# MoSS — Modular Sensory Stream for Integrating Physical Feedback in Vision-Language-Action Models

**arXiv:2604.23272** · KAIST + RLWRLD + SNU (J. Lee, H. Jang, Koo, J. Park, Shin) · Apr 2026

**One line.** The only work here that integrates **two different physical modalities at once** — end-effector tactile and joint-level torque — and shows that existing single-modality methods **get worse** when you give them both.

## 1. What "tactile" means here — two measurement locations

The taxonomy is by *where the signal is measured*, which is the right axis:

- **End-effector sensing** — tactile / force at the gripper or hand
- **Joint-level sensing** — arm torque

Their motivating example makes the complementarity concrete: *"during plug insertion, tactile feedback at the fingertips helps recognize grasp stability, while torque feedback from the arm provides crucial cues for detecting contact and correcting misalignment."* Different signals for different sub-problems of the same task.

## 2. The problem — and the figure that establishes it

Existing approaches *"typically focus on a single type of physical signal"* and are **not designed to grow with sensory complexity**. Naïvely extending them to multiple physical modalities *"does not consistently yield complementary performance gains."*

Average success across four contact-rich real tasks, as modalities accumulate:

| Method | Tactile only | Torque only | **Both** |
|---|---|---|---|
| Base VLA (GR00T N1.5) | — | — | 20.8 |
| Tactile-VLA | 30.2 | — | **20.9** |
| Force-VLA | — | 33.3 | **21.9** |
| TA-VLA | — | 37.5 | **27.1** |
| **MoSS** | 34.4 | 42.7 | **49.0** |

Read the "Both" column. Every existing method **collapses back toward the vision-only baseline** when given two physical modalities — Tactile-VLA falls from 30.2 to 20.9, essentially undoing its own gain. Only MoSS goes up (34.4 / 42.7 → **49.0**), i.e. actually combines them.

This is a genuinely new failure mode in the survey. [[at-vla]], [[restacvla]] and [[tactile-wam]] document tactile *vs vision* interference; MoSS documents tactile *vs torque* interference — physical modalities crowding each other out.

## 3. Model

Three components, each targeting one aspect of the failure:

**(a) Decoupled modality streams.** New streams appended to the **action expert** module (not the VLM backbone), processing physical inputs and exchanging information with pretrained parameters via **joint cross-modal self-attention**. Each modality gets its own stream, so adding a third does not require redesigning the second.

**(b) Two-stage training.** Stage 1 **freezes the pretrained VLA** and trains only the physical-signal streams, letting them *pre-align* their representations with the existing representation space. Stage 2 fine-tunes end to end.

**(c) Future physical signal prediction** as an auxiliary objective, so the model internalises interaction dynamics rather than merely reading current values.

## 4. Results

**Ablation** (success %):

| Method | Unstack Cup | PnP Egg |
|---|---|---|
| **MoSS (full)** | **54.2** | **66.7** |
| w/o decoupling streams | 33.3 | 50.0 |
| w/o two-stage training | 37.5 | 58.3 |
| w/o future prediction | 45.8 | 58.3 |

Decoupling is the biggest single factor (−20.9 on Unstack Cup); two-stage training is worth **+16.7**; future prediction **+8.4**.

**The future-prediction qualitative result** is one of the more satisfying observations in this survey: *"MoSS anticipates contact by predicting rising tactile signals, which prompts increased gripping force."* The prediction is not decorative — it changes the grip before contact arrives. Predictions are shown to track actual signals most accurately precisely at the moments they matter for task success.

**Efficiency** — reported, which most papers skip:

| Configuration | Latency (ms) | Relative |
|---|---|---|
| GR00T N1.5 | 21.0 | 1.00× |
| + MoSS (tactile) | 22.4 | 1.06× |
| + MoSS (torque) | 21.9 | 1.04× |
| + MoSS (both) | 23.4 | **1.11×** |

**+2.4 ms for two additional physical modalities.** Modularity is cheap here because the streams sit on the small action expert, not the large VLM.

The paper also includes an unusually specific impact statement recommending controlled environments, **hardware-level force limits**, and human supervision for high-risk tasks.

## 5. What it adds that the others don't

**Scalability across physical modalities**, and the measurement that establishes it is needed. The "Both" column of Figure 1 is the finding: the field's single-modality integration methods are not additive, and three separate published methods degrade when given a second physical signal. MoSS's answer — one stream per modality, appended to the action expert, pre-aligned while the backbone is frozen — is the only architecture here designed to accept a third and fourth. Read alongside [[ftp-1]], which solves the analogous heterogeneity problem *across sensors* rather than *across modalities*.
