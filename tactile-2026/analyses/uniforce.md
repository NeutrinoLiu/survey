# UniForce — A Unified Latent Force Model for Robot Manipulation with Diverse Tactile Sensors

**arXiv:2602.01153** · King's College London + Imperial + UCL + Bristol (Z. Chen, Ni, Luo, Z. Wu, X. Zhang, Spyrakos-Papastavridis, Jamone, Lepora, Deng, S. Luo) · Feb 2026

**One line.** Gets **force-paired tactile data for free** by exploiting Newton's third law: in a two-finger grasp with *different* sensors on each finger, the opposing forces are equal — so no external F/T sensor and no force labels are needed at all.

## 1. What "tactile" means here — force, disentangled from appearance

The thesis: *"bridging this gap requires disentangling sensor-invariant contact force from sensor-specific tactile signals."*

Three heterogeneous sensors spanning vision-based and **non-vision-based**:

| Sensor | Principle | Form | Output |
|---|---|---|---|
| **GelSight** | optical | flat | high-resolution RGB |
| **TacTip** | optical, biomimetic | **curved**, internal pins | shear/force-sensitive images |
| **uSkin** | **magnetic** (Hall effect) | distributed | multi-channel signals, high frequency |

They differ in *"sensing principles, signal modalities (images vs multi-channel signals), form factors (flat vs curved), and mechanical properties (soft vs hard elastomers)"* — and policies trained on one *"do not directly transfer to other sensors or even to the same sensor with different skins."*

## 2. The data trick — force equilibrium as free supervision

The core insight, and the most economical idea in the cross-sensor cluster:

> *"During a quasi-static bilateral grasp with two fingers equipped with heterogeneous tactile sensors, physics constrains the opposing contact forces to match in magnitude, even though the resulting tactile measurements can differ substantially across sensors."*

Direct **sensor–object–sensor** interactions therefore yield **force-paired tactile signals with no external F/T sensor and no force labels**. Static equilibrium *is* the label.

Compare the alternatives in this survey: [[taf-vla]] built an automated rig with a real F/T sensor to get 10M pairs; [[n0-twam]]'s NeoForce needs a calibrated estimator. UniForce needs neither — just two different fingers and a rigid object.

## 3. Model — inverse and forward dynamics

Two coupled mappings over a **unified marker-image representation**:

- **Encoder / inverse dynamics** — tactile image → latent force
- **Decoder / forward dynamics** — latent force → tactile image, **conditioned on a reference image** (the sensor's own no-contact appearance)

Trained with **force-equilibrium** and **image-reconstruction** losses.

The reference-image conditioning is what carries sensor identity, leaving the latent free to carry only force — the same non-contact/contact decomposition [[unitac]] arrives at, used here for invariance rather than for controllable generation.

The stated contrast with prior work is that image-to-image translation approaches (Touch2Touch, Touch-to-touch translation, TransForce) rely on **location-paired** data and translate between domains *"without explicitly grounding representations in contact dynamics"*, so their latents entangle force with sensor-specific appearance. Against **GenForce** (Nature Communications 2026), which transfers force labels between sensors but *"requires per-target diffusion-based image translation, force-model training, and material compensation"*, UniForce learns one universal encoder end to end.

## 4. Results

- **Force estimation** — consistent improvements over prior methods across GelSight, TacTip and uSkin.
- **Zero-shot transfer** — the pretrained encoder plugs into downstream tasks, with a task head trained on data from **a single sensor**, then transfers to others **without retraining or finetuning**.
- **Force-aware policy learning** — effective cross-sensor coordination in a **VTLA model on a robotic wiping task**.

The deployment story is the cleanest in the cross-sensor cluster: pretrain the encoder once with label-free equilibrium data, train one downstream head on whichever sensor you happen to have, deploy on any of them.

## 5. What it adds that the others don't

**A physical invariant as the alignment target.** [[htt]] and [[tactx]] align sensors by contrastive pairing — the latent means "same contact event"; UniForce's latent means **force**, a quantity with units. That grounding is what lets it span vision-based and non-vision-based sensors in one space, and it makes the representation interpretable in a way learned alignments are not.

And **force equilibrium as a label-free supervision signal** is the kind of idea that should propagate: it turns the robot's own embodiment into a calibration rig. Read alongside [[taf-vla]] (same alignment target, obtained by instrumentation) and [[n0-twam]] (same physical-units strategy, applied at foundation scale).
