# ResTacVLA — Feeling the Unexpected: Contact-Rich Manipulation via Residual Tactile Representation

**arXiv:2607.03387** (v3, Jul 2026) · UCAS + Institute of Automation CAS + Zhongguancun Academy + Dexmal (P. Zhang, B. Xie, Meng, X. Guo, Hao, Deng, Cheng, T. Wang) · Jul 2026 · [site](https://awilekong.github.io/ResTacVLA/)

**One line.** Names the failure **modality collapse** and borrows **predictive coding** from neuroscience: don't feed the policy tactile, feed it *the part of tactile that vision failed to predict*.

## 1. What "tactile" means here — a residual, not a reading

The pathology first. When tactile is projected into a shared feature space as one more modality, *"the high-bandwidth, continuous visual stream naturally overshadows the event-driven, temporally sparse tactile signals."* Trained jointly, VLAs learn to disregard the quiet cues in favour of the loud ones.

The biological argument is the paper's organising idea, and it is a good one. The brain generates top-down predictions of expected sensory states and **attenuates predictable inputs**, attending only to deviations. Their illustration: *humans cannot tickle themselves* — the brain predicts the sensory consequence of self-generated movement and suppresses it, prioritising unexpected external stimuli.

So tactile is reformulated as a **Residual Tactile Representation**: a quantitative measure of *surprise relative to visual priors*. Filtering out visually predictable dynamics **transforms sparse tactile into dense, high-value information gain**, and thereby dissolves the bandwidth mismatch rather than fighting it.

The critique of contrastive visuo-tactile alignment (VITaL, Beyond Sight) follows directly: maximising inter-modal mutual information exploits shared information but **"inadvertently suppresses the critical residual"** — exactly the part that matters.

## 2. Model

Built on **PaliGemma** with a flow-matching action head.

- **Cross-Modal Predictor (CMP)** — distils the discrepancy between visual expectation and physical reality into the residual representation.
- **VQ bottleneck** → **Latent Contact Primitives**, a discrete codebook of physical events vision fails to perceive (unexpected collisions, snap-throughs).
- **Surprise-Aware Gate (SAG)** — uses the **uncertainty of the visual prior** as the neural surprise analogue, amplifying the tactile pathway during visually ambiguous phases and suppressing tactile noise in free-space motion.

## 3. How tactile enters the model

Three-step: predict tactile from vision → take the residual → quantise it → gate its injection by visual uncertainty.

Two design consequences worth separating:

- **Discretisation** into Latent Contact Primitives makes the residual a small vocabulary of *events* rather than a continuous stream, which suits a signal that is genuinely event-driven.
- **Gating on visual uncertainty**, not on contact probability. Every other gate in this survey ([[dream-tac]]'s frame difference, [[feelworld]]'s predicted contact, [[omnivta]]'s contact probability, [[unitacvla]]'s T-CoT) asks *is there contact?* ResTacVLA asks *is vision trustworthy right now?* — a different and arguably more direct criterion for when touch should take over.

The task-phase visualisation (pre-contact → alignment → screwing → approach → success) shows the surprise signal spiking at contact-critical transitions.

## 4. Experiment setup

**Five real-world contact-rich tasks**: precision insertion, threaded assembly/screwing, surface wiping, and others. Baselines include standard VLA and **naive tactile fusion** strategies — the latter being the necessary control for the modality-collapse claim. Robustness is tested against **dynamic physical disturbances**.

## 5. Does it work?

Up to **86.7%** task success, and **+34.6%** average improvement over baselines, outperforming both vision-only VLAs and naive tactile fusion, with superior robustness to dynamic disturbance.

The comparison against **naive tactile fusion** is the load-bearing one: it isolates the residual formulation from the mere presence of tactile input. That naive fusion underperforms is consistent with [[tactile-wam]]'s finding that unrestricted tactile injection is worse than none, and with [[vt-wam]]'s effective-contact-ratio analysis of why.

## 6. What it adds that the others don't

A **principled account of why tactile gets ignored** and a fix that follows from it. The predictive-coding framing turns the sparsity of touch from a problem to be compensated into the *definition* of the useful signal: if vision could have predicted it, it carries no information gain, so discard it. Two mechanisms are reusable independent of the framing — **vector-quantised Latent Contact Primitives** as an event vocabulary, and **gating on visual uncertainty** rather than on contact. Compare [[restacvla]]'s sibling designs: [[vitar]] bounds tactile's influence architecturally, [[tacmamba]] decouples it temporally, ResTacVLA decorrelates it informationally.
