# ViHaTeleop — A Low-Cost, Lightweight Visual-Haptic Teleoperation System for Dexterous Manipulation Learning

**arXiv:2608.16572** · Tohoku University (F. Zhu, Lai, Maestre, Hashimoto) · Aug 2026 · [site](https://laiyanhou.github.io/ViHaTeleop-website/)

**One line.** Closes a specific market gap — *"expensive systems (>$10k) provide haptic feedback... while affordable systems (<$1k) lack haptic feedback entirely"* — with a **$550, 0.7 kg** rig, and runs a **9-participant controlled study** to measure whether cheap vibrotactile feedback actually helps.

## 1. Definition worth adopting

*"We define **contact-critical tasks** as tasks where (i) success depends on detecting contact onset and maintaining stable contact, and (ii) errors are strongly coupled to interaction forces (e.g. slip, breakage, jamming, or insertion failure). Visual constraints are treated as an evaluation context rather than part of the definition."*

That last sentence is a useful correction to much of this literature, which conflates "contact-rich" with "occluded."

## 2. Hardware — itemised, and cheap

| Component | Role | Cost |
|---|---|---|
| HTC Vive Ultimate (VSLAM+IMU) | wrist pose tracking | — |
| Fisheye camera, 160° FOV | hand configuration, wrist-mounted | **$11** |
| LRA rings ×3 (thumb, index, middle) | finger-wise vibrotactile feedback | **$1.4 each** |
| **Total** | | **$550**, **0.7 kg** |

Deployed on **Franka + LEAP Hand + 9DTact**, with the wrist-mounted camera moving with the hand (ACE-like) *"but with significantly reduced mass and torque."* Design choices include **LED illumination**, the fisheye hand camera, and **tactile-aware retargeting constraints**.

## 3. Results — a proper human study

**Nine participants, six contact-critical tasks, matched with/without-haptic conditions**, in both real and simulated environments.

- **Success rates improved on all tasks: +2.2 to +15.6 percentage points.**
- **Completion time effects were task-dependent** — reported as such rather than spun.
- **Subjective ratings** showed significant gains in **contact clarity** and **grasp confidence** in both sim and real (**Wilcoxon signed-rank, p < 0.05**).

The honesty is notable: success gains are described as *"descriptive"*, the time results are mixed, and the statistically significant findings are the subjective ones. A more promotional paper would not have separated these.

**Downstream validation:** visual-tactile policies trained on the collected demonstrations, with tactile benefiting contact-critical subtasks — **peg-in-hole +17 percentage points over vision-only.**

They also integrate a **lightweight depth-camera-based tactile proxy in Isaac Sim**, giving a full pipeline from multimodal demonstration collection to visual-tactile policy training in simulation.

## 4. What it adds that the others don't

**The controlled human-subject study.** [[haptile]] asserts that haptic feedback improves demonstration quality; ViHaTeleop *tests* it, with matched conditions, nine participants and non-parametric statistics — and finds the effect is real but modest on success, significant on operator experience, and task-dependent on time. That is the honest version of a claim the data-collection cluster has been making on intuition.

The **$550 price point with per-finger vibrotactile** also matters practically: at that cost, haptic-informed collection stops being a specialised-lab capability. Compare [[tamen]] (AR-based tactile-visualised recovery teleoperation) and [[art-glove]] (rigid instrumented surfaces) — three points on the cost/fidelity curve for putting contact back in the human's hands.
