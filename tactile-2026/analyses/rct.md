# RCT — A Robot-Collected Touch–Vision–Language Dataset for Tactile Generalization

**arXiv:2606.31694** · ScaDS.AI TU Dresden + LASR Lab TU Dresden (J. He, Färber, **Calandra**) · Jun 2026 · [site](https://faerber-lab.github.io/RCT/) · [code](https://github.com/faerber-lab/RCT)

**One line.** Shows that **the standard evaluation protocol for tactile retrieval is broken** — frame-random splits put near-duplicate frames from the same physical press in both train and test — and audits the field's most-used benchmark to prove it is not a hypothetical.

## 1. The methodological finding

The observation is simple and devastating: **frames within one robot press are not independent samples.** As the sensor indents a material, *"successive tactile frames differ by only a small depth step (0.10 mm in our setup) and are near-duplicate observations of the same physical interaction."*

A frame-random split therefore *"can place frames from the same contact sequence in both training and test. A model may then retrieve a near-duplicate contact observation rather than learn a representation that transfers to new materials or contact conditions."*

Their conclusion: **the independent unit in tactile evaluation is the robot press, not the individual frame.**

## 2. Dataset

| | |
|---|---|
| Tactile frames | **29,279** |
| Materials | **122** industrial reference materials |
| Categories | **7** |
| Sensors | **3× DIGIT** |
| Contact positions | multiple per material |

Each **full contact sequence** is preserved — the ordered frames from one press, one material, one sensor, one contact position — with metadata enabling **held-out evaluation across materials, categories, sensors, contact positions, and contact sequences.** Paired with visual and language annotations.

## 3. The measurements

**Effect of the split, encoder held fixed:**

| Split | tactile→text Recall@1 |
|---|---|
| Frame-random | baseline |
| **Contact-sequence held out** | **−17.7 pp** |
| **+ materials held out** | **−42.0 pp further**, leaving **25.1 ± 6.1%** (3 draws) |

So roughly 60 points of apparent retrieval performance is attributable to split leakage plus material memorisation. Held-out-material retrieval at **25%** is the number the field should be reporting.

**The external audit is the part that generalises.** They examine the released **TVL/HCT** split — used by TVL, UniTouch, and much of the tactile-language literature — and find *"every test contact sequence appears in training"*, with a **training-free nearest-neighbour baseline on raw tactile pixels recovering the correct sequence in 98.3% of cases.**

Raw pixels. No learning. 98.3%. That is not a tactile representation benchmark; it is a near-duplicate detection task.

The authors are careful about scope: *"This does not invalidate touch–vision–language learning; it shows that frame-random retrieval scores should not be interpreted as tactile generalization unless contact-sequence and material overlap are controlled."*

## 4. Constructive findings

- **Uniform sampling beats dense sampling.** Dense per-frame sampling *"adds many near-duplicate observations to contrastive learning"*; uniformly sampling a small number of frames from shallow to deep improves contrastive training.
- **RCT-trained embeddings improve material separability and category recognition on unseen materials.**
- **A remaining failure:** binary hard/soft prediction *"remains close to the majority baseline, even with summary force–depth features."* Their reading — *"human labels carry subjective variation, and scalar summary features through a linear probe may not capture the curve-shape cues most informative of hardness on unseen materials"* — points at the same information the [[geotlm]] result concerns: shape of the response, not its magnitude.
- **Evaluation design note:** because visual and language annotations are material-level rather than per-frame, *"strict one-to-one tactile→vision retrieval is not appropriate; material-level multi-positive evaluation is required."*

## 5. What it adds that the others don't

A **protocol correction with an audit to back it**. Almost every representation paper in this survey reports tactile retrieval numbers on frame-random splits — [[htt]], [[anytouch2]], [[unitac]], [[ht-bench]] included — and RCT establishes that those numbers are inflated by an amount comparable to the entire reported effect size. Its recommendation should be adopted wholesale: **contact-sequence-aware and held-out-material evaluation as default reporting.** Read alongside [[tacverse]] (cross-sensor generalisation collapses) and [[printtacbench]] (classifiers learn 3D-print defects): three independent demonstrations that tactile benchmarks measure the wrong thing unless the splits are designed against it.
