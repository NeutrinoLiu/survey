# TactX — Learning Shared Tactile Representations Across Diverse Sensors

**arXiv:2606.31236** · UC San Diego + Seoul National University + Amazon FAR (J. Park, Bhadang, Sferrazza, Yi, X. Wang) · Jun 2026 · [site](https://tactx-project.github.io/)

**One line.** Aligns **three transduction modalities** — resistive, magnetic, vision-based — into one latent by mounting **different sensors on different fingers of the same gripper**, so every grasp yields a physically paired observation.

## 1. What "tactile" means here — three transduction principles

Not three sensors of one type, but three fundamentally different ways of measuring contact: **optical deformation**, **magnetic-field change**, **resistive pressure response**. The paper's positioning is explicit — prior cross-sensor work *"has primarily focused on aligning sensors within a shared sensing substrate, typically the vision-based family, where signals share a common image-like representation."*

TactX aligns them **in latent space without any per-sensor input transformation** — no resampling into a common image format, unlike the "map signals to a unified input format" line.

## 2. Data curriculum — pairing by gripper geometry

The collection trick is simple and elegant: **mount a different tactile sensor on each finger of the same gripper**. Every quasi-static grasp of a rigid symmetric object produces one measurement from each sensor **of the same physical contact**, temporally aligned.

Because more than two sensors cannot be mounted at once, data is collected **per sensor pair**, and encoders are **trained jointly across all pairs**, inducing a globally consistent latent from pairwise supervision. The authors note this extends naturally: a new modality connects to the shared space through paired contact data with any existing sensor.

*Rigid symmetric* objects are essential here — symmetry is what makes the two fingers' contacts comparable.

## 3. Model

Modality-specific encoders → shared latent Z, trained with:
- **Contrastive alignment** — pull paired contacts from different sensors together
- **Self- and cross-reconstruction** — preserve object- and contact-level structure

The two objectives target the known failure of pure contrastive alignment: it can collapse to whatever is shared while discarding contact detail (the same trade-off [[anytouch2]] measures between static alignment and dynamic perception).

Representation-level validation is by **sensor-identity prediction** and **object classification** in the latent — checking both that sensors are aligned *and* that object information survives.

## 4. Results

**Same-sensor (in-domain), average over three train-test evaluations:**

| Task | Vision | + Tactile GT (oracle) | + TactX |
|---|---|---|---|
| Pick & place | 8.33 | 9.33 | **10.00** |
| P&P (OOD colour) | 6.67 | 8.00 | 7.33 |
| Insertion | 4.00 | **7.33** | 6.00 |
| Wiping | 4.33 | **8.33** | 7.33 |
| Reorientation | 8.00 | 9.33 | **9.67** |

Raw tactile is treated as an **oracle upper bound** — the honest framing, since the goal of a shared latent is to *approach* raw-sensor performance, not exceed it. TactX retains most of it.

**Cross-sensor zero-shot transfer** — the main result: **27.5% → 45.9%** average success over vision-only transfer, across all transfer directions and four tasks (pick-and-place, plug insertion, board wiping, object reorientation), with ACT policies trained on one sensor and deployed on another **without retraining**.

**The binary-contact baseline is the right control** and it fails informatively. Transferring only a contact/no-contact bit gives *"only limited benefit"* on the contact-rich tasks (wiping, reorientation) where TactX gains most — so the shared latent carries **richer contact geometry than a threshold**. And it has a practical flaw the authors report: three sensor-specific thresholds held fixed across all tasks produce **threshold mismatch, higher variance, and inconsistent results**. Anyone tempted by "just binarise the contact signal" should read that.

**Transfer is asymmetric, and the direction is informative.** The weakest case is **eFlesh (magnetic) → FlexiTac (resistive)**, where all methods do badly; the reverse is stronger. The interpretation: *"a policy trained with the lower-dimensional magnetic signal may not learn to use the finer spatial structure available from the resistive sensor at deployment,"* whereas *"policies trained with a richer tactile representation perform more gracefully when deployed with a lower-bandwidth sensor."*

That is a genuinely useful practical rule — **train on the richest sensor you have, deploy on whatever you must** — and it complements [[htt]]'s finding that spatial contact layout carries information a wrench cannot, and [[at-vla]]'s opposite-looking result that low-dimensional tactile grafts better onto a pretrained VLA.

## 5. Stated limitations

- Alignment requires **paired contact under comparable object poses and contact locations** — harder for asymmetric or geometrically complex objects, where two sensors may differ *because of placement* rather than modality.
- Data is **quasi-static gripping**, giving clean supervision but missing dynamic contact variation. Consistent with this, wiping transfers well overall but *"failures can occur under large shear changes or sustained sliding contact."*
- Future work: weakly paired, unpaired, or self-supervised alignment, and dynamic interactions.

## 6. What it adds that the others don't

**Three transduction modalities in one latent, aligned by hardware rather than by format conversion.** The different-sensor-per-finger collection method is the cheapest paired-data trick in this survey and generalises to any new sensor. The asymmetric-transfer finding, and the demonstration that a binary contact bit is not a substitute for a learned latent, are both concrete results the cross-sensor cluster ([[htt]], [[ftp-1]], [[unitac]], [[tacverse]]) can build on.
