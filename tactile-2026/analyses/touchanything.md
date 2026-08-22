# TouchAnything / EgoTouch — Bimanual Tactile Estimation from Egocentric Video

**arXiv:2605.13083** · Harbin Institute of Technology Shenzhen + Meituan Academy of Robotics + Tsinghua SIGS (J. Zhou, Z. Gao, Hong, Z. Liu, G. Zhang, Dai, Zhen, Lyu, H. Wu, Mao, X. Wang, Y. Jiang, Ding, S. Yang) · May 2026 · [site](https://jianyi2004.github.io/TouchAnything-Website/)

**One line.** Identifies **occlusion** as the central obstacle to predicting touch from egocentric video — the hand hides its own contacts — and answers it by adding **wrist-mounted cameras** that see the contact interface directly.

## 1. The problem

The data asymmetry is familiar: egocentric video scales, tactile hardware does not. But TouchAnything names the specific technical obstacle that [[egotac]] and [[felt]] both list as a limitation:

> *"Contact regions are often hidden by the hand itself or the manipulated object, making tactile signals only partially observable from the head-mounted view. This missing contact evidence introduces substantial ambiguity, especially in complex manipulation scenarios."*

Prior datasets *"either rely on single-view capture or focus on relatively narrow interaction settings such as hand-surface contact or single-finger pressing."*

## 2. Dataset — EgoTouch

| | |
|---|---|
| Tasks | **208** manipulation |
| Episodes | **1,891** |
| Frames | **2.1 M+** |
| Duration | **20 hours** |
| Objects | **1,000+** |
| Environments | diverse **indoor and outdoor** |

Synchronised streams: **head-mounted egocentric RGB + dual wrist-mounted RGB**, **bimanual 3D hand pose**, tracker poses, and **continuous pressure maps** from wearable tactile sensors.

## 3. Model

**TouchAnything** — a multi-view vision-to-touch baseline using the egocentric view as primary input, with **cross-view fusion** and **view dropout** so it can *"flexibly leverage available wrist-mounted views at inference time"*. That design choice matters practically: the model degrades gracefully to ego-only when wrist cameras are unavailable.

## 4. Results

Adding wrist views over egocentric-only: up to **+5.0% relative Contact IoU** and **+6.1% relative Volumetric IoU**.

The qualitative figure is the argument: *"the egocentric view suffers from occlusion, while wrist-mounted views reveal the contact interface... ego-only prediction misses contact in occluded regions, whereas multi-view prediction recovers accurate pressure distributions consistent with the ground truth."*

The relative gains are modest in aggregate but concentrated exactly where they should be — **contact localisation and pressure estimation under egocentric occlusion**.

## 5. Stated limitations

- **Glove appearance bias** — all training data uses tactile gloves, which *"may introduce glove-specific appearance bias and limit generalization to bare-hand tactile estimation."* Glove-to-bare-hand retargeting is proposed. This is a real problem for the whole human-video-tactile line: the model may be reading the glove, not the hand.
- **Not saturated** — Contact IoU and Volumetric IoU *"continue to improve with more training data, suggesting that vision-to-touch prediction remains data-hungry"*, matching [[egotac]]'s monotone scaling curves.
- Benchmark is tactile *estimation* only; downstream uses (contact-aware manipulation, grasp stability, affordance, tactile world models) are future work.

## 6. What it adds that the others don't

**A hardware answer to the occlusion problem.** [[egotac]] and [[felt]] both identify occlusion as the failure mode of vision-to-touch and propose additional viewpoints as future work; TouchAnything builds the multi-view dataset and quantifies the gain. The view-dropout design also makes it deployable at either camera configuration. Read as the third member of the vision-to-touch cluster: [[egotac]] maximises scale and in-the-wild transfer, [[felt]] optimises for closed-loop robot deployment, TouchAnything optimises for bimanual human interaction under occlusion.
