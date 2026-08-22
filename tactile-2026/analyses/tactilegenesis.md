# Tactile Genesis — Exploring Tactile Sensors at Scale for Learning Dexterous Tasks

**arXiv:2606.22332** (v2, Jul 2026) · Carnegie Mellon University + **Genesis AI** (T. Chung, Yamazaki, Patel, Duburcq, Qiao, Fragkiadaki, Nayebi) · Jun 2026 · [site](https://neuroagents-lab.github.io/tactile-genesis/)

**One line.** Runs the experiment no lab can afford in hardware — **every tactile sensor type, placement and resolution on the same task** — and finds that **placement dominates sensor type, and resolution barely matters at all.**

## 1. The experiment that motivates the simulator

*"Each sensor effectively defines a new robot, and no lab can replicate the same learning experiment across all of them."* A lab that buys one sensorised hand cannot repeat its dexterous learning experiment across capacitive arrays, magnetic skins, vision-based elastomers, strain gauges, contact microphones and multisensory fingertips.

The question posed is therefore not whether touch helps but **which tactile abstraction a policy actually needs** — and *"when richer tactile fields justify their hardware cost."*

## 2. Platform

**Seven contact abstractions under one interface:**
- binary contact
- raw contact depth
- **per-taxel kinematic force/torque**
- elastomer marker displacement
- geometry-aware proximity
- **contact audio**
- **voxelized temperature field** — *"the first of its kind in robot learning physics simulation platforms"*, simulating contact heat transfer, diffusion, generation and radiation

With **configurable placement, resolution**, and a **realistic noise model: drift, hysteresis, dead taxels, crosstalk** — the actual failure modes of real taxel arrays, which most tactile simulators omit entirely.

**Scale:** past **20,000 parallel environments** and **1,000 taxels** on a single GPU — **3–20× throughput** over previous tactile simulators. Works on any robot surface (XHand1, Inspire, Allegro, Shadow, Franka, Wuji, SharpaWave).

## 3. Findings — and they contradict current hardware practice

Teacher-student policies on three dexterous tasks, ablating **sensor type, placement, resolution and noise**, with transfer verified on the real **XHand1**.

1. **Proprioception alone is insufficient on every task.**
2. **Placement dominates sensor type.** *"Fingertip-only coverage trails whole-hand coverage by a wide margin, while adding the palm and proximal phalanges closes most of the gap to the privileged teacher."*
3. **Resolution matters far less than coverage.** *"Placing 200 taxels across the whole hand suffices across tasks."*
4. **Per-taxel force/torque is consistently the most useful sensor type.**

The practitioner guidance is stated directly, and it is a genuine challenge to the industry:

> *"First, **cover the palm and proximal phalanges before paying for higher-end fingertip sensors**. Second, prefer per-taxel force/torque as a default abstraction. Third, consider a proximity channel when the task involves capturing an object that approaches the hand... The first two findings are **at odds with the de facto convention in current commercial tactile fingertips, which concentrate spatial resolution on the distal pad and stop there**."*

A further inference for simulator design: *"the dominant source of useful tactile information for these tasks is the **coarse spatial distribution of contact** rather than the fine-grained mechanics of the substrate. This is encouraging for rigid-body tactile simulation, since substrate physics is exactly what is most expensive to model faithfully."*

## 4. Stated limitation — and it is the important one

*"Because our students distill from a privileged teacher, they inherit its strategy and are therefore bounded by it."* The comparison measures **what the teacher can be matched on**, not what touch is capable of. The proposed fix — curiosity-driven RL with tactile as the only sensory channel — would separate the two.

## 5. What it adds that the others don't

**A controlled sensor-design study**, which is impossible in hardware and which nobody else has run. Its three headline findings — coverage beats resolution, force/torque beats other abstractions, palm and proximal phalanges beat fingertip-only — are actionable hardware guidance derived from policy performance rather than perception benchmarks, and they cut against how commercial tactile hands are currently built. The temperature and contact-audio channels also open modalities the rest of this survey does not touch at all.

Read against [[taco-bench]], which reaches the compatible real-world conclusion that spatial resolution does not determine manipulation success, and [[htt]], which finds the opposite for a specific task requiring spatial regrip layout — the reconciliation being that *coverage* is spatial information too.
