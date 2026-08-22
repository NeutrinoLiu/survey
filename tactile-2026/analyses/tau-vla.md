# τ — Learning Touch-Augmented Vision-Language-Action Models from Future Visual Supervision

**arXiv:2607.24485** (v3, Aug 2026) · Beijing Jiaotong University + BIGAI (Cheng, J. Xu, W. Li, Y. Chen, Gao, Y. Wang, Peng, Han) · Jul 2026 · [site](https://cocacola-lab.github.io/tauPage/)

**One line.** Supervises tactile representation learning with **future visual feature change**, JEPA-style — the tactile encoder is trained to predict what the *camera* will see next, and the whole predictive branch is deleted at inference.

## 1. What "tactile" means here

Bilateral vision-based tactile (left/right fingers), encoding **normal and shear deformation**:

```
z^touch_t = [ E_tou(T^L_t) ; E_tou(T^R_t) ]
```

with `E_tou` **shared across both fingers and initialised from the π₀.₅ vision encoder** — reusing the pretrained visual pathway rather than training a tactile-specific encoder, on a limited data budget.

The gap identified: existing methods handle *instantaneous* contact state, or model temporal dynamics via **6D wrench sequences**, which capture structure-transmitted force-torque variation but "lack dynamics-aware representation for high-dimensional vision-based tactile signals with fine-grained spatial contact patterns."

## 2. Data curriculum

**TacAura** — synchronised vision, proprioception, and vision-based tactile across four contact-rich tasks: **plug insertion, USB insertion, stamp press, whiteboard erasing**. Released with teleoperation tools and data-conversion utilities as a complete collection infrastructure.

Zero-shot generalisation is tested on two axes per task: **two unseen objects** (differing in appearance and surface texture) and **two unseen distractor sets** (added objects and visual clutter).

## 3. Model

Three components on a **pretrained π₀.₅ backbone**:

1. **π₀.₅ VLA** — observation `o_t = {I_t, ℓ̃_t}` where `ℓ̃_t = [ℓ_t ; q_t]` concatenates task description with proprioceptive state; action expert predicts chunk `â_{t:t+H}` by conditional flow matching.
2. **Tactile encoding and adaptation module** — maps tactile into the VLA's semantic space via a touch encoder + touch adapter, producing tactile tokens fused alongside visual and textual tokens in the LLM.
3. **JEPA-style predictive branch** — training only.

## 4. How tactile enters the model — and how it is *trained*

Tactile tokens join the LLM context alongside visual and textual tokens (the very placement [[n0-vtla]] argues against). The novelty is not the placement but the **auxiliary objective**:

Conditioned on the **current tactile representation and the subsequent action sequence**, a predictor forecasts **the change in future visual features**. The target is the feature change between current and future observations as encoded by the **VLA's own vision encoder**, detached. A semantic similarity objective aligns prediction with target.

The distinction from vanilla JEPA is stated precisely: JEPA predicts target representations from contextual observations for general representation learning; here the branch predicts **future visual feature change from an action-conditioned tactile representation**, to learn *control-relevant* interaction dynamics.

Two properties make it attractive: it operates **purely in latent space** (no pixel reconstruction — the objective [[tactile-wam]] shows is misaligned with contact anyway), and it is **removed at inference**, so representation quality improves at zero deployment cost.

## 5. Experiment setup

Four real tasks, each decomposed into **three stages** with per-stage success (e.g. Grasp / Align / Insertion; Pick / Contact / Press) — a reporting choice that makes the ablations far more diagnostic than a single number.

Three τ variants differing in which view supervises the JEPA branch: **τ-FrontSup**, **τ-WristSup**, **τ-DualViewSup**.

## 6. Does it work?

**Main result:** τ exceeds the strongest baseline by **35, 20, 20, and 45 percentage points** on the four tasks.

**The supervisory view matters, task-dependently.** τ-FrontSup is best on plug insertion (65%) and USB insertion (50%) — front-view features give better spatial cues for target-hole localisation. τ-WristSup is best on stamp press (90%) and whiteboard erasing (95%), where the wrist view is less occluded. And **τ-DualViewSup underperforms the best single-view variant**, even falling slightly below ForceVLA† on stamp press — the authors attribute this to the difficulty of learning complementary signals from two views under limited data with a simple fusion strategy.

**Ablations** (per-stage success; final-stage average in the last column):

| Variant | Plug (G/A/I) | USB (G/A/I) | Stamp (P/C/P) | Whiteboard (P/C/W) | Avg |
|---|---|---|---|---|---|
| **τ** | 100/75/**60** | 100/50/**40** | 100/100/**90** | 100/100/**95** | **71.25** |
| w/o action-seq conditioning | 100/75/55 | 100/45/30 | 100/100/75 | 100/100/75 | 58.75 |
| w/o predictive SSL | 100/75/50 | 100/50/25 | 100/100/70 | 100/100/60 | 51.25 |
| w/o tactile module (= π₀.₅) | 100/50/20 | 100/35/20 | 100/70/35 | 100/100/40 | 28.75 |

The stage decomposition earns its keep. Removing the **predictive SSL** costs 20 points on average — but **intermediate alignment and contact success rates are unchanged**; only the final execution stage degrades. The predictive objective helps the model capture *temporal evolution during sustained contact*, not contact detection. Same pattern for action-sequence conditioning: grasping, picking and contact stages are largely unaffected while final execution drops, with the largest hits on stamp press and whiteboard erasing — the two tasks most dependent on precise force control.

Removing the tactile module entirely (= plain π₀.₅) collapses even the *alignment* stage (75→50 on plug, 100→70 contact on stamp), so touch contributes at two different places for two different reasons.

**Generalisation** is sharply task-dependent:

| Task | In-distribution | Unseen objects (avg) | Unseen distractors (avg) |
|---|---|---|---|
| USB insertion | 40% | 30% (25 / 35) | **22.5%** (20 / 25) |
| Whiteboard erasing | 95% | 90% (90 / 90) | **95%** (no degradation) |

USB insertion is sensitive to both, and **more sensitive to visual clutter than to object change** (−17.5 points) — visual clutter breaks target localisation and precise alignment. Whiteboard erasing is essentially invariant to both. The honest reading: touch generalises well for *sustained surface contact* and poorly for *precise geometric alignment*, where the binding constraint is vision.

**Stated limitations.** Compute constraints prevented comparison against tactile-aware **world models** — so τ is not benchmarked against [[dream-tac]], [[vt-wam]] or [[tactile-wam]]. Cross-task, cross-embodiment and cross-sensor transfer are unexplored.

## 7. What it adds that the others don't

The **inverted supervision direction**. Every other predictive-tactile paper here supervises tactile prediction with future *tactile*; τ supervises the tactile encoder with future *visual* feature change, conditioned on actions. That sidesteps the pixel–contact misalignment problem entirely and needs no tactile-specific decoder — and since the branch is training-only, it costs nothing at deployment. The three-stage success reporting also makes it one of the few papers that can say *which* part of the task its contribution fixes.
