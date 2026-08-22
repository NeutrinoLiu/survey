# SoftVTBench — A Deformation-Aware Visuo-Tactile Dataset and Benchmark for Deformable-Object Manipulation

**arXiv:2608.18701** · Tuojing Intelligence · Tsinghua · KCL · Imperial · CMU · HKU · Beihang (Jing, Wang, Hao, … Yu) · Aug 2026 · [code](https://github.com/TuojingAI/SoftVTBench) · [site](https://softvtbench.github.io/)

**One line.** The most methodologically careful tactile evaluation of 2026: it separates what the policy can *see* from what the evaluator can *measure*, and in doing so shows that most published tactile gains are not identifiable.

## 1. What "tactile" means here

Two disjoint streams, and the separation is the whole point.

**Policy-visible touch** — dual-finger **tactile RGB plus marker motion** (GelSight-style optical tactile with tracked markers), recorded at **20 Hz** alongside multi-view RGB, proprioception, language, and actions. This is what a policy is allowed to condition on: contact appearance and local shear.

**Evaluator-only physical state** — a **finite-element (FEM) representation** of each deformable object as a volumetric mesh, tracking nodal positions under contact. After removing rigid-body transport, the residual nodal displacement measures how much the object actually changed shape. The policy never sees this.

The authors state the rationale sharply: *"tactile observations cannot serve as their own evaluation target."* Touch tells the policy how contact is evolving; the hidden FEM state tells the evaluator what that contact did to the object. Everything downstream follows from refusing to conflate the two.

Built in Isaac Sim + Isaac Lab.

## 2. Data curriculum

| | |
|---|---|
| Demonstrations | **4,000** expert |
| Tasks | 40 pick-and-place, across 4 diagnostic suites |
| Assets | **50+**, incl. 10 volumetric deformables and matched rigid twins |
| Rate | 20 Hz, all streams synchronised |

The **2 × 2 design** is the design contribution:

| Suite | Object type | Variation axis | Demos |
|---|---|---|---|
| Object-Soft | deformable | object identity | 1,000 |
| Spatial-Soft | deformable | spatial layout | 1,000 |
| Object-Rigid | rigid twin | object identity | 1,000 |
| Spatial-Rigid | rigid twin | spatial layout | 1,000 |

Every deformable object has a **rigid twin with matched mesh, texture and mass** but negligible grasp-induced deformation, reusing the same layout and language instruction. A rigid–soft difference therefore isolates deformability from task difficulty.

**Calibration happens before any policy is trained.** Scripted physical probing sweeps gripper closure over the FEM deformation measure to fix an object-specific tolerance τ_o and a gripper envelope 𝒢_o = [g_min, g_max]. The criterion is thus calibrated independently of the policies it later scores — a discipline almost nothing else in this survey observes.

**Splits.** ID evaluation: 500 held-out initial states per suite (50 per task). OOD: **nine single-factor conditions** on the deformable suites only — dome light 135 → {180, 270, 67.5}; object mass × {1.25, 1.75, 2.5}; Young's modulus × {0.8, 0.5, 2.0}. Each condition perturbs one factor while reusing the ID reference's task, initial state and seed.

Curation: automatic integrity checks (completeness, alignment), a VLM semantic metadata check, then human verification (episode review, label spot-checking, calibration consistency, coverage audit).

## 3. Model

SoftVTBench trains nothing of its own. It evaluates three deliberately different policy families:

- **Diffusion Policy** — trained from scratch, *not* language-conditioned.
- **π₀.₅** — an adapted vision-language-action model.
- **FastWAM** — a world–action model.

Choosing one from each family is what lets the paper claim its findings are about tactile fusion in general rather than one architecture.

## 4. How tactile enters the model

Per-policy, in whatever way that family natively supports — the benchmark's contribution is not a fusion mechanism but the **control** for it. The critical design decision is that every demonstration **stores both gripper-action encodings**, binary and continuous. That allows the 2 × 2 cross:

`VO-B / VO-C / VT-B / VT-C` = {vision-only, visuo-tactile} × {binary, continuous gripper}.

Without this, "tactile input" and "finer actuation" are confounded — and as the results show, they very often are.

## 5. Experiment setup

Closed-loop rollouts in simulation. Two metrics, deliberately decoupled:

- **TSR** — Task Success Rate, completion only.
- **DSR** — Deformation-aware Success Rate: credits an episode only if it completes the task **and** keeps peak normalised deformation R_max within the pre-calibrated tolerance.

Their difference is, by construction, **exactly the share of successes that were physically unsafe**.

## 6. Does tactile actually help?

The honest answer the paper reaches is: *sometimes, mostly under distribution shift, and far less than the literature implies.*

**Finding 1 — completion hides damage.** Across **all 12** in-distribution deformable configurations there are rollouts that complete the task while exceeding the deformation tolerance, accounting for **0.7–24%** of each configuration's successes. A completion-only protocol scores these as wins.

**Finding 2 — deformability's cost is not intrinsic.** Rigid vs. soft twins, TSR (%):

| Model | Input | Object: Rigid | Object: Soft | Spatial: Rigid | Spatial: Soft |
|---|---|---|---|---|---|
| Diffusion Policy | VO-C | 40.0 | 37.4 | 14.0 | 15.6 |
| | VT-C | 35.0 | 40.0 | 11.0 | **33.0** |
| π₀.₅ | VO-C | 60.0 | 41.6 | 50.4 | 26.0 |
| | VT-C | 59.6 | 41.4 | 54.0 | 27.6 |
| FastWAM | VO-C | 64.0 | 62.0 | 25.0 | 37.0 |
| | VT-C | 61.6 | 57.6 | 30.0 | **56.4** |

π₀.₅ loses 18.4 points from rigid to soft on object variation; Diffusion Policy and FastWAM lose 2.6 and 2.0, inside protocol resolution. On spatial variation the sign *reverses* — FastWAM scores 12.0 points **higher** on deformables.

**Finding 3 — the one every tactile paper should read.** Matched sensing–control ablation for π₀.₅ on Object-Soft:

| Config | TSR | DSR |
|---|---|---|
| VO-B | 30.2 | 27.2 |
| VO-C | 41.6 | **38.4** |
| VT-B | 41.0 | 28.0 |
| VT-C | 41.4 | 35.0 |

From VO-B, **continuous control alone buys +11.4 points; tactile alone buys +10.8; both together buy nothing extra** (41.4%). A study comparing VO-B against VT-C would credit touch with a gain that finer actuation reproduces on its own. And VO-C vs. VT-B differ by 0.6 points of TSR but **10.4 points of DSR** — 7.7% of VO-C successes leave the safety zone versus **31.7%** for VT-B. The tactile policy succeeds just as often and squeezes far harder.

**Finding 4 — touch buys robustness, not peak performance.** Pooled over the nine OOD conditions, VT-C beats VO-C on TSR in **6/6** policy–suite comparisons and on DSR in **5/6** (one-sided exact sign test, p = 0.016 and p = 0.11). In-distribution the same comparison splits 4–2, consistent with chance. Notably π₀.₅ VT-C degrades by only −0.4 TSR under shift versus −5.8 for VO-C, and FastWAM VT-C *gains* +1.4 DSR on Object-Soft.

But the paper refuses the clean story: under shift, touch **widens** the TSR − DSR gap for the two policies whose gap was already large (Diffusion Policy 2.6 → 6.2 points on Object-Soft; π₀.₅ 2.6 → 6.8). FastWAM is the constructive counter-case, holding TSR and DSR within one point everywhere.

**Is DSR real?** The authors check it three ways. *Discriminative*: on Object-Soft, TSR ranks Diffusion Policy VT-C above VO-C (40.0 vs 37.4) while DSR reverses it (30.4 vs 33.6). *Attainable*: none of the 2,000 accepted deformable demonstrations exceeds tolerance — median R_max = 0.433, 95th percentile 0.713 — so violations are not inherited from supervision. *Achievable*: FastWAM leaves only 0.7–6.5% of its successes outside the safety zone.

## 7. What it adds that the others don't

The **evaluator-only channel**. Every other tactile benchmark either exposes touch to the policy or measures physical consequence — SoftVTBench is the only one doing both on the same timeline, which is what makes "the policy squeezed too hard but finished the task" an expressible outcome. Combined with rigid twins and stored dual action encodings, it is the only 2026 resource that can *falsify* a tactile gain rather than merely report one. Its own summary is the most useful sentence in the 2026 tactile literature: **making touch available does not by itself ensure effective multimodal fusion.** Compare [[tactidex]], which arrives at a strict tactile-aware success criterion by a different route.
