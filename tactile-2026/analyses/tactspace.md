# TactSpace — Learning a Physics-enriched Shared Latent Space for Tactile Sim-to-Real Transfer

**arXiv:2606.18959** · ETH Zürich (Robotic Systems Lab, Micro- and Nanosystems Lab, ETH AI Center) + NVIDIA (Joarder, Bhardwaj, Zurbrügg et al.) · Jun 2026

**One line.** Gives up on simulating the sensor's raw signal at all — instead aligning **simulated penetration depth** and **real capacitance** in a shared latent, *"eliminating the need for accurate raw-signal simulation while preserving relevant contact information."*

## 1. The idea

Rather than closing the sim-to-real gap in signal space, close it in **latent** space. Modality-specific encoders project physically dissimilar observations — simulated penetration depths and stresses on one side, real capacitive readings on the other — into a common embedding trained with:
- **self- and cross-reconstruction**
- **contrastive alignment**

encouraging *"modality-invariant yet information-rich representations."*

The choice of real modality is notable: **capacitance**, not a gel image. Simulated depth and real capacitance have no pixel correspondence whatsoever, which makes this the most extreme representation mismatch bridged in the survey.

## 2. Multi-physics as extra modalities

The "physics-enriched" part: the simulator contributes **multiple physical modalities** — penetration depths from a rigid-body simulator, **stresses from FEM** — and aligning against both yields better embeddings than either alone.

## 3. Results

Trained **exclusively in simulation**, tested directly on real sensor measurements, across three tasks: **indenter shape identification, force prediction, geometric reconstruction**.

Zero-shot sim-to-real transfer across physically dissimilar representations, and incorporating multi-physics modalities gives:
- **−16.7%** force prediction error
- **−45.8%** shape reconstruction error

The shape-reconstruction improvement is large, and its direction is sensible: FEM stress fields carry geometric information that penetration depth alone flattens.

Also released: *"an efficient **Warp-based implementation of a penalty-based tactile simulation model for Isaac Lab**, enabling scalable tactile data generation."*

## 4. What it adds that the others don't

**Sim-to-real by latent alignment rather than signal fidelity**, applied across genuinely incommensurable modalities (depth ↔ capacitance). It is the simulation-side counterpart to [[tactx]] and [[htt]]'s cross-sensor latents, and the natural complement to the fidelity-maximising approaches: if you can align latents, you do not need the simulator to produce anything resembling the sensor's output — you only need it to produce something *causally equivalent*. The finding that adding a second physics modality (stress on top of depth) substantially improves transfer also suggests simulators should expose multiple derived fields rather than one rendered signal.
