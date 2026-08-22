# DYNA-2 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Dyna-2: A 1-Million-Hour Scaling Law for World-Action Models |
| **Org** | Dyna Robotics |
| **Date** | 2026-08-10 |
| **Artifact** | dyna.co/dyna-2 (`page.html` / `page.txt`) + PR Newswire release (`pr.html`) |
| **Disclosure level** | **B — detailed company research page.** No peer-reviewed paper; methodology and ablations are described in depth, but the corpus, code, and weights are not released. Pipeline internals ("our internal quality bar") are named but not specified numerically. |
| **Corpus** | **>1,000,000 hours** egocentric human video (~170 years of continuous waking experience). **Zero robot data in pretraining.** |
| **Stance** | *Scale the one source that already exists at unbounded scale, and deliberately do not clean the embodiment gap away.* |

## 1. Thesis — the source-selection argument

Dyna-2 frames data curation as answering three questions in sequence:
1. What is the right *source* of pretraining data for robot learning?
2. Does scaling that source produce a scaling law on robot performance?
3. Which modeling/objective choices are required for that law to hold?

Their answer to (1) is derived from the goal rather than from convenience:

> *"a general purpose robot should eventually be capable of doing any economically valuable task that is currently being done by humans. The right source of pre-training data, therefore, ought to be sensorized recording (e.g., video) of humans performing those very tasks, which already exists at effectively unbounded scale."*

Teleoperation and specialized capture devices are acknowledged and used, but rejected as the *scaling* substrate: *"each hour of that data has to be deliberately produced, which bounds how far it can carry pre-training on its own."*

## 2. Sources & scale

- **>1M hours** of human manipulation video, mostly **head-mounted first-person** recordings of everyday manipulation — cooking, tidying, folding, assembling.
- Collected by **data partners** plus **internal operation**.
- Reported corpus descriptors: clips, unique task instructions, distinct objects (rendered as live counters on the page).

## 3. The cleaning pipeline

Stated as a four-function pipeline, applied *"to ensure validity and consistency across the various sources"*:

```
data cleaning → hand-pose extraction → validation → filtering
```

**The gate:** *"For episodes that pass the hand-pose quality bar, their annotations contain 3D hand-pose tracks."*

**Pseudo-action derivation from hand pose:**
| Robot quantity | Derived from |
|---|---|
| End-effector trajectory | **wrist poses** |
| Continuous grasp signal | **thumb–index aperture** |

This is a minimal, two-channel action abstraction — deliberately far less rich than EgoScale's 22-DoF retargeted hand or Qwen-RobotManip's full SE(3)+width parameterization. Simplicity here is what allows a single representation to survive a million heterogeneous hours.

## 4. The deliberate *non*-cleaning decision

The most distinctive curation choice in this survey:

> *"We do **not** perform any visual or embodiment-specific data processing to reduce the visual or kinematic gap between the pre-training data and the downstream robot data; our interest is to study model properties from scaling, and scaling alone. Therefore, we believe our findings are general and should also transfer to other embodiments not studied in this work."*

No hand inpainting, no robot rendering, no retargeting to target morphologies, no human–robot alignment, **no co-training with robot data at all**. Compare Qwen-RobotManip, whose entire H2R engine exists to erase exactly this gap. Dyna-2's claim is that at sufficient scale the gap closes on its own — and their ablations are designed to test precisely that.

## 5. The two-tier corpus — quality bar as a *router*, not a filter

The single most transferable idea here. Because *"not all human activity recording setups can extract hand poses that pass our internal quality bar"*, and *"running large-scale annotations on millions of hours of data is in itself a huge infrastructure challenge and creates a lag between the amount of total data versus amount of data that is action-labeled and quality-controlled"* —

> *"it's safe to assume that in the regime of pre-training from human data, there will always be a large amount of un-annotated human video data."*

So episodes that **fail** the hand-pose bar are **not discarded**. They are routed to the **video-prediction objective only**, where no action labels are needed:

| Tier | Gate | Objective |
|---|---|---|
| **Action-labeled** | Passes hand-pose quality bar | video loss **+** action loss |
| **Video-only** | Fails the bar / not yet annotated | video loss only |

Dyna-2 then shows this reject tier is not merely salvage — **it is its own scaling axis** (see §7). Every other pipeline in this survey treats a failed quality check as deletion.

## 6. Scaling-experiment data hygiene (unusually rigorous)

Three controls that make the scaling claims falsifiable:

1. **Nested, exact-hours subsets.** Rungs at exactly **1,000 / 10,000 / 100,000 / 1,000,000 hours**, with *"identical proportion from each data source"* kept at every rung. *"a larger budget never exchanges data, only adds it, and differences between points on the scaling-law curves cannot be explained by distribution shift between subsets."* — this rules out the most common confound in data-scaling studies.
2. **Fixed disjoint validation set.** A separate **100-hour** human validation set, disjoint from all training subsets, held fixed across every evaluation.
3. **Metric-robustness by construction.** Citing the "emergence is a mirage" critique, they report **four metrics** — MSE, L1, and thresholded **accuracy@τ for τ = 0.5 and 0.1** — because *"nonlinear or discontinuous metrics can unintentionally generate apparent emergence from smoothly improving models."* τ=0.5 measures general motion intent (right for cross-embodiment transfer); τ=0.1 measures precision (right for in-domain trends). Checkpoint bias removed by averaging **10 checkpoints in a late-step window**.

Held-out robot evaluation set: **39 tasks on two distinct stationary bimanual YAM platforms** — 12 from an internal benchmark, **27 from the external xdof ABC dataset**, deliberately included *"to make sure the evaluation is not biased toward the tasks we design ourselves."* No checkpoint trained on a single trajectory from either.

## 7. Scaling evidence — three distinct laws

**(a) In-domain (human→human) scaling law up to 1M hours.** All four metrics improve monotonically and are *"well described by a power law in hours."* accuracy@0.1 rises **51%** across the ladder vs **12%** for MSE.

**(b) The human→robot *transfer* scaling law — the headline claim.** Evaluated zero-shot on the 39-task held-out robot suite, with no adaptation or fine-tuning: *"all metrics rank monotonically with the scale of the pre-training human data. To our knowledge this is the first scaling law demonstrated across the embodiment gap."* An **inflection point between 10k and 100k hours** suggests *"with sufficient coverage, cross-embodiment knowledge transfer may emerge from just scale alone."*

**(c) It survives post-training onto real robots.** 14 tasks, 3 embodiments (bimanual YAM + parallel jaws; YAM + WUJI-2 20-DoF dexterous hands; semi-humanoid prototype), ≤10 h robot data per task, **identical post-training recipe, robot data only, no co-training, no alignment**, blind evaluation by people not involved in model development, 10–12 trials per task.

| Pretraining budget | Mean normalized score (14 tasks) |
|---|---|
| 1k h | 20% |
| 10k h | 28% |
| 100k h | 45% |
| **1M h** | **53%** |

Threshold effects: **Lockbox Key Turning was never solved at ≤100k h; at 1M h it succeeded 90% of the time.** Data efficiency: **Bottle Cap Untwisting was post-trained on ~10 minutes of robot demonstrations**, climbing 10% → 50% purely with pretraining scale. Language following (Targeted Drink Retrieval) 58% → 83%.

## 8. What actually makes the transfer law emerge (the objective ablation)

Fixed action-data budgets at 5k / 50k / 100k hours, three recipes, all evaluated zero-shot on the 39-task suite at matched steps:

| Recipe | Result |
|---|---|
| **Action-only** (no world modeling) | Loses to joint on **39 of 39 tasks at every scale**; *"severe and unpredictable overfitting patterns as data scales"* |
| **Joint** (action + video on the same data) | Beats action-only everywhere, but **does not scale with data** |
| **Video co-training** (joint + extra *action-unlabeled* human video for the video loss) | **The only recipe that improves as action data grows.** No advantage at 5k h; the gap widens with scale |

**Video-only data as an independent axis.** Holding action-labeled data fixed at 50,000 h and scaling video-only data 0 → 1k → 10k → 50k h produced monotonic improvement on held-out robot metrics; repeated an order of magnitude up (250k h action + 0/250k/750k h video-only) with the same result. Notably, scaling video-only data *"has no impact and perhaps slightly worsens human data evaluations"* while improving robot ones — i.e. **the video axis buys cross-embodiment generalization specifically, not general accuracy.**

This retroactively justifies §5: the rejected, unannotated tier is where the transfer capability comes from.

## 9. Limitations & disclosure caveats
- **No numbers published for the quality bar**, source composition, partner identities, or rejection rates.
- No compute/model-size scaling — *"We leave compute and model-size scaling experiments for future work."*
- No corpus, code, or weights released; claims are not independently reproducible.
- Post-training deliberately omits human-robot alignment/co-training, which they concede *"would likely yield better post-training performance"* — so absolute numbers understate what the recipe could do.
- Marketing framing in the PR ("first true scaling law in robotics") is stronger than the research page's own careful claim; the page's claim is specifically about *cross-embodiment transfer* scaling.

## 10. Transferable takeaways
1. **Treat the quality bar as a router, not a filter.** Rejects feed a weaker-supervision objective instead of the bin — and here they turn out to carry the generalization.
2. **Nested exact-hours subsets with preserved source proportions** is the correct way to build a scaling ladder; anything else confounds scale with distribution shift.
3. **Report thresholded *and* continuous metrics** so a scaling claim cannot be a metric artifact.
4. **Include an external held-out evaluation set** you did not design.
5. **A minimal action abstraction** (wrist pose + thumb–index aperture) is what makes a million heterogeneous hours representable at all.
6. Read against **Ψ₀** (800 h clean, no scale) and **EgoScale** (20,854 h, fitted log-linear law). Dyna-2 is the extreme end of the volume-over-purity bet, and the only one with a published cross-embodiment law.
