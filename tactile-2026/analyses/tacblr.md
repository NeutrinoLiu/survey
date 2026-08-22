# SBLR — Tactile Sim2Real without Tactile Simulation via Bottlenecked Latent Reconstruction

**arXiv:2608.15897** (v2) · University of Michigan + Google DeepMind (F. Yang, J. Yu, Wi, Fazeli, Berenson) · Aug 2026

**One line.** Same conclusion as [[ptld]] — skip tactile simulation — but reached with **unpaired random-play data** and rectified flow, and it beats a physics-based tactile simulator by 7.5–15 points.

## 1. The motivation

*"Robot sensor designs, particularly tactile sensors, are highly diverse and evolve rapidly. Modeling each sensor in simulation demands substantial domain expertise and computational approximations can degrade the fidelity of the simulated signals."*

The obsolescence argument is the sharpest version of this case anywhere in the survey: *"tactile sensor designs evolve rapidly, often **outpacing the engineering effort needed to build and validate the sensor simulations**."* By the time a simulator for a gel is validated, the gel has been superseded.

## 2. Method

**(1) Train on a simulator-native oracle sensor** that is *"easy to construct without modeling any particular sensor"* — here, **point cloud + fingertip forces**. No gel, no optics, no elastomer.

**(2) Align real sensor latents to oracle latents at inference.**

**Two-stage policy training:** the policy first learns from oracle latents, then **bottlenecked latent reconstruction** adapts it to *"the information loss expected when using the real sensor instead of the oracle."* That second stage is the key idea — it does not pretend the real sensor carries as much as the oracle, it *trains the policy to work with less*.

**Alignment** is learned from **unpaired random-play data** collected in both simulation and the real world, using **rectified-flow transformation networks trained on nearest-neighbour pseudo-pairs** — the same machinery [[tactalign]] uses for human-to-robot tactile transfer, applied here to sim-to-real.

## 3. Results

**Simulation**, three contact-rich tasks: SBLR *"matches or approaches the performance of an oracle with direct access to tactile simulation."*

**Hardware**, Peg Insertion and Gear Meshing with **GelSight Mini and DIGIT**: **85–97.5% zero-shot success** *"without requiring any sensor-specific modeling or calibration"*, outperforming a **physics-based tactile simulation baseline by 7.5–15%**.

Beating an actual tactile simulator while never simulating tactile is the result that matters — it suggests the residual sim-to-real error in physics-based tactile simulation currently exceeds the information loss from routing through an abstract oracle.

## 4. What it adds that the others don't

**Unpaired alignment plus explicit modelling of information loss.** [[ptld]] needs an instrumented cell and paired demonstrations to distil; SBLR needs only **random play in both domains**, which is far cheaper and requires no privileged tracking at deployment or training. The bottlenecked-reconstruction stage is also a more honest formulation than plain distillation: the real sensor genuinely carries less than the oracle, and the policy should be adapted to that gap rather than assumed to bridge it.

Together with [[ptld]], it makes a serious case that the simulation cluster's central premise — that faithful tactile simulation is a prerequisite — may be avoidable.
