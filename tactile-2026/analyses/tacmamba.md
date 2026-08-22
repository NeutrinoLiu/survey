# TacMamba — A Tactile History Compression Adapter Bridging Fast Reflexes and Slow VLA Reasoning

**arXiv:2603.01700** · Zhejiang University + GigaAI + HIT (Z. Wang, Y. Wang, Ren, P. Li, Y. Liu, Nie, Long, Ye, X. Wang, Zhu, Dong) · Mar 2026

**One line.** Frames the tactile-VLA problem as **Kahneman's System 1 / System 2 split** and solves the frequency mismatch with a Mamba state-space model that compresses 100 Hz tactile history into a constant-size hidden state queried asynchronously by a ~1 Hz planner.

## 1. What "tactile" means here

Deliberately low-dimensional: **1-D force streams at 100 Hz** from a custom fingertip.

The hardware is part of the contribution: a **modular morphology-based tactile fingertip** whose integrated compliant contact body supports both fingertip and fingerpad interaction and **mechanically projects distributed contacts onto a single-axis force sensor**. FEM characterisation reports peak contact pressure vs. applied load for both contact modes, plus deformation, von Mises stress and contact-pressure fields. A reconfigurable dual-clamp interface attaches it to generic parallel grippers.

The choice is defended explicitly and its cost acknowledged: sufficient for isolating high-frequency temporal dynamics without image-processing overhead, but **prevents extraction of fine-grained spatial topology**.

## 2. The problem, stated as a frequency mismatch

- Tactile perception needs **high-frequency processing with long-horizon memory** (System 1)
- Visual policies operate at **low control frequency** (System 2)
- Transformers on 2 s of 100 Hz history generate hundreds of tokens at **O(N²)** — unviable above 100 Hz
- LSTMs give fast inference but **catastrophically forget** over thousands of timesteps

Plus two training bottlenecks: end-to-end co-training of high-frequency tactile into a large VLA is computationally prohibitive and risks forgetting semantic priors; and **critical interaction events are under 5% of a trajectory** (a button snap-through), so uniform sampling drowns them in reaching motion.

Their critique of existing tactile VLAs is architectural: ForceVLA, OmniVTLA and similar feed tactile tokens **directly into the heavy VLA backbone**, so sensory processing is bottlenecked by the VLA's inference speed. They can only use **instantaneous tactile snapshots at drastically reduced frequency**, discarding micro-slips and snap-through vibrations.

## 3. Model

**System 1 — Mamba Tactile History Compressor.** A selective state-space model streaming at **100 Hz**, recursively updating a hidden state `h_t`, with **O(1) inference latency measured at 0.45 ms**. Linear-time sequence modelling with a compressed hidden state resolves the memory–latency trade-off that neither Transformers nor LSTMs can.

**System 2 — frozen VLA** (π₀.₅) at ~1 Hz.

The two run **asynchronously and fully decoupled** — this is the design's core.

## 4. How tactile enters the model

The compressed hidden state `h_t` is projected and **asynchronously injected as a soft prompt** into the VLA. Because it is a soft prompt on a compressed state rather than a token stream, integration is **plug-and-play without joint pre-training** — the VLA never sees raw tactile and is never retrained end-to-end with it.

**Tactile-Guided Dual-Stage Training:**
1. **Self-supervised pre-training via Ternary Temporal Discrimination** — the tactile encoder independently learns causal contact dynamics by discriminating temporal ordering.
2. **VLA fine-tuning with Phase-Uniform Sampling** — sampling uniformly over *interaction phases* rather than timesteps, so the <5% of frames containing the critical contact event are not drowned out.

## 5. Experiment setup

Tasks chosen for **visual ambiguity**, where touch is the only ground truth: **sequential button pressing** (discrete counting — how many clicks have registered), **implicit state switching / counting**, and **deformable clothes folding**. Baseline: visual-only **π₀.₅**.

## 6. Does it work?

**100% success rates** on discrete counting and implicit state switching, against a failing visual-only π₀.₅ baseline, while **strictly satisfying hard real-time constraints** (100 Hz loop, 0.45 ms encoder latency).

The mechanism the authors identify for the baseline's failure is worth noting: π₀.₅ exhibits **temporal overfitting to static frames**. In a counting task, the visual state before and after a click is nearly identical, so a frame-conditioned policy has no way to represent "how many times have I already pressed". The tactile *history* — not the tactile *reading* — carries the state.

Phase-uniform sampling is credited with improving data efficiency "multiple times", and the system is reported to maintain fine-grained precision without overshoot under occlusion and long horizons.

**Stated limitations, honestly scoped.** (1) A **1-D force sensor**, not a GelSight — no spatial topology. (2) The evaluation *"deliberately targets specific edge cases (e.g. severe occlusion), comparing primarily against π₀.₅"* rather than exhaustive benchmarking, so the 100% figures should be read as demonstrations on tasks constructed to require temporal tactile memory, not as general superiority. Future work: feeding GelSight image sequences directly into the Mamba network.

## 7. What it adds that the others don't

**Frequency decoupling with a constant-size state.** Every other design in this survey either downsamples touch to the policy rate ([[unitacvla]], [[touchworld]] partially) or runs a separate reactive controller on the *current* reading ([[omnivta]], [[unitacvla]]). TacMamba is the only one that keeps a **lossless 100 Hz history** available to a 1 Hz planner at O(1) cost, and the only one whose target tasks — counting clicks — cannot be solved from instantaneous contact at all. Phase-uniform sampling is a small, reusable fix for a data imbalance that afflicts every contact-rich dataset in this survey.
