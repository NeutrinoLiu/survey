# EgoTac — In-the-wild Tactile Prediction from Egocentric Vision

**arXiv:2608.15060** · Shanghai Qi Zhi + SJTU + Tsinghua + Fudan (W. Zhang, C. Yuan, Z. Zhang, Cheng, Y. Gao) · Aug 2026

**One line.** Asks whether the enormous existing corpus of egocentric human video can be **retroactively labelled with touch** — and answers yes well enough to produce force traces on Ego4D and EPIC-KITCHENS clips recorded years before anyone thought about tactile.

## 1. What "tactile" means here — force and contact on a shared hand topology

Two supervision types unified:
- **Continuous force** (Newtons) from pressure-instrumented gloves
- **Binary/dense contact** labels from hand-object interaction datasets

All annotations are mapped onto a shared **MANO** hand representation, which is what makes eight heterogeneous sources with *"differing spatial coverage"* trainable together.

The asymmetry motivating the whole paper: *"Unlike vision, which can be recorded passively and remotely, touch must be measured at the physical contact interface, making data collection sensitive to sensor placement, calibration, spatial coverage, durability, and wearability."*

Their critique of existing HOI datasets is worth flagging for anyone using them: most *"rely on **analytical** contact labels inferred computationally from mesh proximity rather than genuine physical contact."* Proximity is not contact — the same conflation [[tactidex]] uses real pressure to filter out.

## 2. Data curriculum — EgoTac-Dataset

**>5.7M image–tactile pairs**, combining the newly collected **EgoTac-SC** (3.9M frames, 2.9K clips, bimanual, glove hand pose, **real force**, in-the-wild) with eight prior HOI datasets. The dataset comparison table is itself a useful artifact: of eighteen egocentric datasets surveyed, only **six** have real force (ActionSense, PressureVisionDB, EgoPressure, OpenTouch, FEEL, EgoTac-SC), and only three of those are in-the-wild.

## 3. Model

A concise **encoder–decoder** taking temporal RGB input and **jointly predicting dense continuous tactile values and contact classification labels** — a mixed force-contact objective that lets heterogeneous supervision train one model.

## 4. Results

**In-domain:** force MAE **< 0.06 N**, contact **F1 > 0.70**.

**Out-of-domain contact** (vs. RGB-based 3D contact estimators BSTRO, DECO, HACO on OAKINK2 and FPHA): EgoTac wins **AUROC, IoU, Precision and F1 on both**. HACO wins Recall in places, but the authors diagnose it correctly — *"this improvement largely stems from over-predicting contact regions, resulting in greater coverage at the cost of more false positives,"* with HACO *"predict[ing] contact even when no interaction occurs."*

**Out-of-domain force on OpenTouch** — and the evaluation protocol here deserves credit. OpenTouch uses a *different tactile sensing system*, and its public annotations *"do not provide the per-taxel physical calibration and effective sensing areas required for a reliable conversion to Newton level forces."* Rather than quietly reporting a Newton-level MAE that would rest on unsupported cross-sensor calibration assumptions, they **normalise both prediction and ground truth to [0,1]** and measure whether *relative* tactile intensity and temporal structure survive domain shift. Result: **force-cosine similarity > 0.7**, **rise/fall F1 > 0.4**, with predicted traces visibly tracking the ground-truth trend.

That is exactly the honest move — and it quantifies the cross-sensor problem from a third angle, alongside [[tacverse]]'s negative R² and [[unitac]]'s chance-level classifier.

**Zero-shot in the wild:** plausible tactile predictions on **EgoDex, EPIC-KITCHENS, Ego4D, EgoPAT3D** and casually self-recorded video. Qualitatively, EgoTac identifies **which hand is actively manipulating**, captures fine-grained contact and force change over long horizons, and **adapts predicted patterns to object geometry**. Attention maps confirm it focuses on hand-object interaction regions.

**Scaling — both axes, both monotonic.**

Data *diversity* (OOD contact on OAKINK2):

| Sources | AUROC ↑ | IoU ↑ | F1 ↑ |
|---|---|---|---|
| 1 dataset | 0.603 | 0.065 | 0.120 |
| 4 datasets | 0.832 | 0.263 | 0.407 |
| 6 datasets | **0.844** | **0.285** | **0.435** |

The 1→4 jump is enormous (AUROC 0.603 → 0.832, F1 0.120 → 0.407) and 4→6 is modest — diminishing but not exhausted. Data *volume* (10% → 100%) also improves AUROC, IoU, F1 on OAKINK2 and force-cosine/rise-fall F1 on OpenTouch.

Note how low the absolute IoU stays (0.285): predicting *where* on the hand contact occurs remains hard even at best.

## 5. What it adds that the others don't

It inverts the data problem the entire tactile field is stuck on. Every other work here answers "how do we collect more tactile data?" — by rigs ([[taf-vla]]), simulation ([[univtac]]), robot-free interfaces ([[tacumi]]), or scale ([[n0-twam]]). EgoTac asks whether the tactile labels can be **manufactured from video that already exists at internet scale**, and its monotone diversity/volume scaling curves suggest the approach has headroom. If it holds up, corpora like Ego4D and EPIC-KITCHENS become tactile datasets retroactively — which would change the economics of everything else in this survey. Compare [[felt]] and [[tacgen]], which generate tactile signals for the same reason but for robot-side sensors rather than human video.
