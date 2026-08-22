# TacFiLM — Tactile Modality Fusion for Vision-Language-Action Models

**arXiv:2603.14604** (v2, Jul 2026) · McGill + Mila + NVIDIA (Morissette, Abyaneh, Chang, Houssaini, Meger, Lin, Tremblay, Dudek) · Mar 2026 · [site](https://charliem7.github.io/projects/TacFilm/)

**One line.** Injects tactile by **FiLM-modulating intermediate visual features** — no extra tokens, no extra sequence length, no cross-attention module — and shows it beats both concatenation and cross-attention on success, force, and time.

## 1. What "tactile" means here

Vision-based tactile (DIGIT / GelSight-style) encoded by a **frozen pretrained tactile representation**, used as a conditioning vector rather than a token.

The problem statement is a token-budget argument: *"the common baseline of feature concatenation often requires training separate tactile encoders and appends additional tokens to the VLA input, increasing sequence length and computational cost while risking performance degradation as context grows."*

## 2. Model

**Feature-wise Linear Modulation (FiLM)** applied to intermediate visual features of a pretrained **OpenVLA-OFT**, conditioned on tactile embeddings. Post-training finetuning only.

The design choice is stated as a deliberate rejection of cross-attention: FiLM *"enables parameter-efficient adaptation without extensive multimodal pretraining"* and, unlike concatenation, **conditions intermediate visual features on tactile embeddings** rather than appending to the sequence — *"this preserves pretrained visual-language priors while incorporating tactile signals."*

## 3. Encoder selection — a genuinely useful side experiment

Before the main experiments, four pretrained tactile encoders are compared on tasks that **isolate tactile signatures relevant to insertion**: Rotation-High and Rotation-Low (is the held peg rotated relative to its initial pose? — contact-geometry sensitivity), Contact (contact vs non-contact — force/deformation features), plus continuous force regression from Sparsh TacBench.

| | T3 | Sparsh-IJEPA | Sparsh-MAE | **Sparsh-DINO** |
|---|---|---|---|---|
| Rotation-High (%) | 92.73 | 99.15 | 99.36 | **99.36** |
| Rotation-Low (%) | 83.09 | 96.44 | 96.64 | **98.42** |
| Contact (%) | 73.31 | 85.08 | 93.92 | **95.39** |
| **Average** | 83.04 | 93.56 | 96.64 | **97.72** |
| Force estimation (RMSE ↓) | 58.64 | 40.27 | 36.61 | **36.09** |

**Sparsh-DINO** wins on all four and is adopted. This is the only place in the survey where a policy paper *selects* its tactile encoder empirically rather than by convention — and the T3 vs Sparsh gap (83.0 vs 97.7 average) is a large, rarely-reported difference between two widely used tactile foundation encoders.

## 4. Experiment setup

Real-robot, **over 1,000 rollouts**, on contact-rich insertion and drawer-opening tasks, in-distribution and out-of-distribution. Baselines: **OpenVLA-OFT** (vision-only), **TactileConcat**, **Cross-Attn**.

Metrics go beyond success: **direct insertion/opening percentage** (did it succeed on the first attempt without retries), **completion time**, and **maximum applied force** — the last being the axis [[softvtbench]] argues completion-only benchmarks miss.

Two deliberate **visual degradation** conditions: **80% dimmed lighting** and a **partially frozen camera stream (50% frame updates)**.

## 5. Results

**In-distribution:** 100% success on the **3 mm clearance Circle-Peg** task, and up to **+50%** over the next-best baseline on selected tasks.

**Out-of-distribution:** 100% on 3 mm clearance peg insertion; **+30%** on HDMI cable plugging; and roughly **one-third of the force** applied by baselines on selected tasks.

**Under visual degradation** (Circle-Peg 3 mm):

| Condition | Method | Success % | Direct % | Time (s) ↓ | Max force ↓ |
|---|---|---|---|---|---|
| 80% dimmed | OpenVLA-OFT | 93.33 | 0.00 | 16.29 ± 9.85 | 73.50 ± 8.41 |
| | TactileConcat | 86.67 | 26.67 | 11.15 ± 9.21 | 78.03 ± 30.16 |
| | Cross-Attn | 53.33 | 0.00 | 9.29 ± 5.96 | **159.96 ± 29.15** |
| | **TacFiLM** | **100.00** | 26.67 | **8.62 ± 2.13** | **67.79 ± 6.25** |
| 50% frames | OpenVLA-OFT | 73.33 | 0.00 | 15.70 ± 12.39 | 113.19 ± 40.39 |
| | TactileConcat | 80.00 | **40.00** | 14.64 ± 13.91 | 71.52 ± 34.51 |
| | Cross-Attn | 46.67 | 0.00 | 7.53 ± 1.25 | **161.10 ± 30.70** |
| | **TacFiLM** | **100.00** | 26.67 | 8.12 ± 1.90 | **51.68 ± 5.33** |

Three things stand out.

**Cross-attention is the worst option here**, dropping to 46–53% success while applying **2–3× the force** of every other method — 160 N against TacFiLM's 52–68 N. It is fast because it slams the peg in. That is a strong caution given how many designs in this survey ([[deco]], [[feelworld]], [[unitacvla]], [[touchworld]]) route tactile through cross-attention; the mechanism is clearly not universally safe under distribution shift.

**TacFiLM's force stability is the standout metric**, not its success rate: 51.68 ± 5.33 N under frozen frames, versus 113.19 ± 40.39 for vision-only. Both the mean and the *variance* collapse — the policy applies consistent, moderate force rather than occasionally very high force.

**TactileConcat achieves a higher direct-insertion rate in one condition** (40.00 vs 26.67) while being slower and forceful, which the authors report rather than hide. Direct insertion and gentle insertion are not the same objective.

## 6. Stated limitations

A broader task set was blocked by a concrete obstacle worth noting: *"without precise visuotactile simulators, extending the task set proved difficult, as the experiments had to be conducted in a real-world setup."* That is the practical case for the simulation cluster ([[univtac]], [[tactilegenesis]], [[etac]]) stated from the demand side. FiLM fusion is also designed around OpenVLA-OFT; extension to π₀.₅ is future work.

## 7. What it adds that the others don't

The **fusion-mechanism comparison under matched conditions** — concatenation vs cross-attention vs FiLM, on the same backbone, sensor, and data — with force and time reported alongside success. Its finding that **feature-level modulation generalises more stably than either alternative**, and that cross-attention degrades catastrophically on force under visual shift, is a concrete design result. The tactile-encoder benchmark (Sparsh-DINO > Sparsh-MAE > Sparsh-IJEPA > T3) is a useful bonus that other policy papers could simply adopt.
