# TacO — Benchmarking Tactile Sensors for Object Manipulation

**arXiv:2605.21976** · UC San Diego + CMU + SNU (Zorin, Buynitsky, Bhadang, Si, Kroemer, Temel, Y.-L. Park, Tolley, Yi, X. Wang) · May 2026

**One line.** Six sensors, four modalities, three tasks, one fixed learning pipeline — and the conclusion that **spatial resolution barely matters for manipulation success**, while sensor *friction* matters a lot.

## 1. What "tactile" means here

Deliberately plural. TacO's whole design is that "tactile" is not one modality:

| Sensor | Modality | Normal (N) | Shear (N) | Taxel size (mm) | Spatial res. | Freq (Hz) | High friction | Price |
|---|---|---|---|---|---|---|---|---|
| FSR | Resistive | 0.2–20.0 | — | 34×34 | 1 | on-demand | ✗ | $5 |
| FlexiTac | Resistive | 0.2–10.0 | — | 2×2 | 12×32 | on-demand | ✗ | $35 |
| eGain (liquid metal) | Resistive | 0.0–27.5 | — | 8.4×12.7 | 2×3 | on-demand | ✓ | $5 |
| eFlesh | Magnetic | 0.0–30.0 | 0.0–17.5 | 0.5×0.5 | — | 20–100 | ✓ | $35 |
| Daimon | Visual | 0.3–30.0 | 0.1–8.0 | 0.1×0.05 | 320×240 | 60/120 | ✓ | $965 |
| Contact mic | Acoustic | — | — | 30×30 | 1 | — | ✓ | $27 |

Note the ~200× price spread and the ~10⁵ spread in spatial resolution. That range is the experiment.

## 2. Data curriculum

Teleoperated demonstrations, collected independently at **two institutions over 2,000 miles apart** on comparable robot platforms — an explicit reproducibility control almost no tactile paper bothers with.

Three tasks chosen to stress different tactile properties:
- **Pick-and-place with unknown object mass** — latent physical property vision cannot see
- **Plug insertion with occluded contact geometry** — vision is blocked exactly when it matters
- **Object reorientation** — continuous force modulation

For every sensor, both a vision-only and a visuotactile policy are trained **on the same data**, with tactile readings simply removed for the vision-only case. That is the per-sensor control.

## 3. Model

**ACT (Action Chunking Transformer)** as a fixed policy backbone — a CVAE predicting fixed-length action chunks, chosen because its transformer encoder accepts arbitrary extra modalities as tokens. Holding the policy constant is what allows attribution to the sensor.

## 4. How tactile enters the model

Sensor-specific encoders, all producing tokens for the ACT encoder:
- **MLP** on raw scalar/array values (FSR, FlexiTac, eGain, eFlesh)
- **Convolutional** encoding of tactile images (Daimon)
- **Spectral processing** of vibrotactile audio (contact microphone)

## 5. Experiment setup

Two orthogonal analyses, which is the methodological contribution:

- **Per-sensor** — vision-only vs. visuotactile, same data → isolates the *tactile signal's* contribution.
- **Cross-sensor vision-only** — compare vision-only policies across sensors → isolates the *embodiment* effect (material, appearance, form factor), since no tactile signal is used at all.

Plus a **hardware repeatability** test using an open-source test kit built from off-the-shelf components.

## 6. Does tactile actually help?

**Pick-and-place** (success rate):

| Method | FSR | FlexiTac | eGain | Contact mic | Daimon | eFlesh |
|---|---|---|---|---|---|---|
| Vision only | 0.50 | 0.75 | 0.50 | 0.65 | **0.95** | 0.85 |
| Vision + tactile | 0.50 | 0.85 | 0.75 | **0.90** | 0.80 | 0.90 |

Note Daimon — the $965 vision-based sensor — **gets worse** with tactile (0.95 → 0.80), with failures involving table collisions. The mechanism the authors identify for why tactile helps at all here is clean: vision-only policies output an *average* gripper width, while tactile policies learn distinct widths for light vs. heavy objects. Gains concentrate on heavy objects.

**Plug insertion and reorientation:**

| | Insertion: FlexiTac | Insertion: Mic | Insertion: eFlesh | Reorient: FSR | Reorient: Daimon | Reorient: eFlesh |
|---|---|---|---|---|---|---|
| Vision only | 0.1 | 0.2 | 0.3 | 0.6 | 0.2 | 0.5 |
| Vision + tactile | 0.3 | **0.7** | **0.7** | 0.8 | **0.7** | 0.8 |

Insertion is where tactile pays: +0.2 to +0.5 absolute everywhere. The failure-mode analysis is more informative than the numbers — vision-only policies misplace the plug, push weakly, release; tactile policies *slide the plug along the socket surface* and often partially insert. ~55% of FlexiTac tactile failures are partial insertions, i.e. the policy got the contact strategy right and lacked recovery demonstrations.

**Finding — spatial resolution doesn't matter.** On reorientation, Daimon (320×240, rich shear) and eFlesh and FSR all land near 80%. Across all tasks the authors find *no significant difference* between high-resolution (FlexiTac, Daimon) and low-resolution (FSR, eFlesh, mic) sensors on manipulation success. Their reading: high-resolution sensors excel at *perception* tasks (localisation, recognition) and that advantage does not transfer to coarse manipulation policy performance. **Access to a task-relevant physical signal beats dense spatial representation.**

**Finding — friction is a confound nobody controls for.** Vision-only success by sensor surface material:

| Material | Pick-and-place | Insertion | Reorientation |
|---|---|---|---|
| Low friction | 0.625 | 0.10 | **0.60** |
| High friction | **0.81** | **0.25** | 0.35 |

The sensor's *skin* changes policy success by up to 20 points **with no tactile signal used at all**. High-friction compliant surfaces generally help — except in reorientation, where controlled slipping is the desired behaviour and high friction causes lifting or jamming. Adding tactile feedback largely erases the difference, i.e. touch lets a policy regulate force well enough to avoid the downside of grippier skin. Any comparison of two tactile sensors that does not control for this is partly measuring rubber.

**Repeatability.** FSR most consistent; FlexiTac drifts at rest; eGain and Daimon show a viscoelastic "training phase"; eFlesh drifts consistently; the mic is inherently noisy. Daimon has the second-lowest variance — and that reliability **does not translate into higher policy success**, so repeatability is not the dominant factor either.

**Stated limits.** Resolution is not isolated on tasks that would actually require fine-grained sensing; vision is retained throughout most executions rather than ablated during tactile-critical phases; and everything is ACT-based imitation learning.

## 7. What it adds that the others don't

The **cross-sensor vision-only control**. It is the only work here that measures how much of a "tactile sensor's benefit" is just its rubber, and the answer is: a substantial fraction. Together with the resolution null result, TacO is the strongest available argument that cheap sensors are enough for manipulation — the $5 eGain and $27 microphone are competitive with the $965 camera-based sensor. Read alongside [[softvtbench]] (which shows control granularity is a second uncontrolled confound) and [[tacverse]] (which shows resolution *does* matter once you ask for force regression).
