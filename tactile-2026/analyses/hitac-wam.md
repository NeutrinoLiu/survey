# HiTac-WAM — A Hierarchical Tactile World Action Model for Contact-Rich Robot Manipulation

**arXiv:2608.19574** · Institute of Automation CAS + ImprintX Robotics + BAAI (Xue, C. Zhang, Ma, Yao, Cui, S. Wang) · Aug 2026

**One line.** Same group as [[feelworld]], one month later, with the hierarchy repurposed: instead of gating fusion, the contact→deformation→slip chain becomes a **scoring function for candidate action chunks**, plus an execution-time reference for detecting when reality diverges from the plan.

## 1. What "tactile" means here

A **directed hierarchy of three factors**, forecast per candidate action:

```
T̂^(k)_{t+h} = ( Ĉ^(k)_{t+h},  D̂^(k)_{t+h},  R̂^{slip,(k)}_{t+h} )
              bilateral      3D deformation   slip risk
              contact        field
```

The dependency structure is physical and explicitly enforced: **interface deformation is meaningful only under contact, and slip risk depends on the resulting contact and deformation states.** Downstream stages are conditioned on **stop-gradient** signals from upstream ones — so the hierarchy shapes the forward computation without letting slip gradients corrupt the contact representation.

The problem statement is sharper than most. When a world action model samples K candidate action chunks from the same visual history with independent noise draws, the paired visual rollouts **all look plausible** — aligned gripper, moving eraser, centred connector — while the actions produce different physical outcomes: missed or unilateral contact, insufficient pressure, slip, lateral jamming. Appearance-based scoring cannot distinguish a successful grasp from a visually identical failure.

## 2. Data curriculum

Three contact-rich tasks: **chip grasping, blackboard erasing, USB insertion**. Observations `H_t = {V_t, q_t, τ_t, ℓ}` — synchronised RGB views, robot state, **bilateral** tactile history, language instruction. Candidate chunks are `A^(k)_t ∈ R^{H×7}`.

## 3. Model

A **tactile branch augmenting a pretrained world action model**, so the visual/action backbone is inherited rather than retrained. The tactile branch predicts `T̂^(k)_{t+1:t+H} = f_θ(H_t, A^(k)_t)`.

Critical alignment property: at forecast offset h, action `a^(k)_{t+h−1}` is **temporally aligned with and conditions** the tactile target at t+h. That same alignment is what later lets the selected forecast serve as a frame-by-frame execution reference.

## 4. How tactile enters the model — a directed attention mask

The design decision, stated plainly: **tactile queries attend to the video–action context of each candidate, while video and action queries are prevented from attending to tactile tokens.**

This is one-way isolation rather than gating. The visual/action stream is protected absolutely — the pretrained world action model's behaviour is unchanged by adding the tactile branch, which is why the cost is so small (video LPIPS +1.4%, action MAE +0.9% versus the tactile-free model). Touch reads everything and writes nothing back into prediction; it writes into **selection** instead.

Contrast the three positions now on the table: [[n0-twam]] (free bidirectional attention, capacity separated in weights), [[feelworld]] (contact-gated, bidirectional only during contact), HiTac-WAM (strictly unidirectional). All three report gains; none has been compared head-to-head.

## 5. Experiment setup — selection, then verification

**Forecast-guided selection.** K candidate action–forecast pairs are ranked by a score `J^(k) = J(A^(k), T̂^(k))` combining the hierarchical tactile forecast with **task-progress estimates** derived from the predicted visual rollouts. The best rollout is executed.

**Online forecast verification.** The selected tactile forecast is retained as an execution-time reference. A deviation `e_t = τ^observed_t − τ̂_t` is monitored; when `e_t > γ` **persistently**, corrective replanning is triggered (safe retreat + renewed candidate generation). The persistence requirement is what keeps this from firing on transient noise.

Baselines: DreamZero-style world action model; single-candidate execution; task-progress-only ranking under a **fixed generation budget** (an important control — otherwise more candidates alone could explain the gain).

## 6. Does it work?

**Prediction quality:** mean contact **F1 = 0.921**, contact onset/release errors of **2.1 / 2.3 frames**, 3D displacement **L2 = 0.058 mm**, slip **AUPRC = 0.247**.

Note that slip AUPRC of 0.247 is low in absolute terms — slip forecasting remains poor even when it is the paper's third hierarchy level, echoing the contact-vs-slip gap in [[feelworld]] (98.1% vs 83.4% F1).

**Does the hierarchy earn its place?** Matched-budget ablations, which is the right control:

| Comparison | Improvement from the directed hierarchy |
|---|---|
| vs. deformation-only predictor | **−17.6%** 3D displacement L2 error |
| vs. slip-only predictor | **+60.4%** slip AUPRC |

The authors state the conclusion carefully: *"shared multi-output prediction alone is insufficient"* — it is the **explicit directed conditioning**, not merely joint prediction, that buys the accuracy. That is the cleanest available evidence for structuring tactile prediction rather than emitting one blob.

**Cost of adding the tactile branch:** video LPIPS +1.4%, action MAE +0.9% relative to the world action model without tactile prediction. Nearly free, which the directed mask explains.

**Real-robot success** (average over three tasks):

| System | Success |
|---|---|
| Single-candidate execution (DreamZero) | 31.1% |
| Task-progress ranking (fixed generation budget) | 35.6% |
| **+ hierarchical tactile forecast selection** | **61.1%** |
| **+ online forecast verification (full)** | **72.2%** |

The fixed-budget comparison is the one that matters: **35.6% → 61.1% with non-overlapping confidence intervals**, at the same number of generated candidates. So the gain is from *scoring candidates by predicted touch*, not from sampling more of them. Verification then adds a further +11.1.

**Verification audit** — unusually specific and honest: **14 trigger cases coincided with execution anomalies, 12 of which were followed by successful completion; the remaining two were false triggers with no effect on outcome.** Of 21 untriggered failures, at most three involved an undetected anomaly. So the detector is high-precision (2 false positives) with a reasonable miss rate.

**Stated limitations.** Forecast quality on *model-generated* action chunks is not evaluated separately (only on logged data), and **predicted slip risk is not calibrated for online detection** — which, given AUPRC 0.247, is the load-bearing caveat.

## 7. What it adds that the others don't

It is the only work here that keeps each tactile forecast **attached to its candidate action** through both selection *and* execution — consulted before acting to rank, retained during acting as the reference reality is checked against. The matched-budget ablation separating "hierarchy" from "joint prediction" is the strongest evidence in this survey that physical structure in the prediction target is worth engineering, and the directed attention mask is the cheapest way anyone has found to add tactile forecasting to a pretrained world action model without disturbing it.
