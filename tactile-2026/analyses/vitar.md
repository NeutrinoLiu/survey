# ViTaR — Visuo-Tactile Residual Adaptation for Foundation VLA Manipulation

**arXiv:2608.15816** · Beijing Institute of Technology (Y. Wang, R. Wu, J. Liu, X. Li) · Aug 2026 · [site](https://icr-lab.github.io/ViTaR)

**One line.** Argues that every other tactile-VLA method makes the same structural mistake — **granting touch unbounded influence over action generation** — and instead confines tactile to *selecting and scaling bounded residuals* on top of a completely frozen VLA.

## 1. What "tactile" means here

Two separate signals with **explicitly separated roles**, which the ablation then validates:

- **`m_i`** — a **marker-derived contact descriptor** (from the marker chunk), used for **residual selection**: *whether and which* correction.
- **`z_i`** — a **tactile summary** (from the tactile-image chunk), used for **continuous gain scaling**: *how much*.

## 2. The argument

A frozen VLA "commits to the same action regardless of whether contact is established, slipping, or lost." Two existing remedies, both rejected:

- **Fusing tactile into VLA internals** — risks catastrophic forgetting of the visual-semantic priors that make VLAs generalise.
- **RL-based adaptation** ([[taccorl]], [[torl-vla]]) — demands exploratory rollouts **at near-failure contact states, where safety margins are thinnest**.

The key observation: *a frozen VLA already provides the correct semantic action direction; what it lacks is calibration of execution under contact conditions absent during pretraining.* Hence tactile should be an **execution modulator**, not a participant in action generation.

The second argument is about the learning signal. Global reward regression is ill-suited because **absolute scores are not comparable across heterogeneous contact states**. Within-state preference comparison (Bradley–Terry) resolves this: ranking corrections **at the same restored contact state** recovers relative local effects without cross-state calibration.

## 3. Model — two stages on a frozen base

**Effect-Guided Modeling (EGM)** — estimates whether the current contact state is **improvable** (adaptability gate) and ranks available residuals by expected local effect, via **pairwise outcome comparisons at restored decision points**. Heads: Effect Head, Gate Head.

**Residual Action Modulation (RAM)** — converts that evidence into a discrete choice (retain base action, or select residual option) plus a **continuous tactile-conditioned gain**. Residual options are structured: X±, Y±, G± (gripper). The frozen VLA action is preserved whenever contact evidence does not justify intervention.

Base policy: **frozen OpenVLA-OFT**.

## 4. How tactile enters the model

Never into the VLA. Marker and tactile-image chunks feed **only** EGM and RAM, which sit entirely outside the frozen policy and operate on a **bounded, discrete residual space** in the base action's own coordinates. The pretrained representations are untouched by construction — no forgetting is possible.

Supervision comes from **short-horizon branch rollouts restored to the same decision point**, i.e. the environment is reset to a shared contact state and different residuals are tried, giving within-state comparisons.

## 5. Experiment setup

**UniVTAC**, seven contact-rich tasks (Insert Tube, Put Bottle, Insert HDMI, Grasp Classify, Lift Can, Lift Bottle, Pull out Key). Plus **three physical-robot tasks** (Insert Hole, Wiping the Board, Lift Bottle), 20 trials each.

Beyond success, the paper reports post-hoc diagnostics that are **never inputs to the deployed policy**: paired recovery, preservation, unsafe/early-termination rate, outcome regret, and best-branch gain.

## 6. Does it work?

**UniVTAC:** ViTaR **61.3%** average versus **30.7%** for the frozen OpenVLA-OFT base — a **+30.6 pp** doubling, improving on **all seven** tasks. Physical robot: **48.3%** average, +30.0 over OpenVLA-OFT, +15.0 over Tactile-VLA, +21.7 over ACT (40.0% Insert Hole, 40.0% Wiping, 65.0% Lift Bottle).

The authors are unusually careful about what this does and does not show. They note ViTaR **does not dominate every task** — π₀.₅ is stronger on Lift Can (70% vs 44%), Tactile-VLA on Lift Bottle (97% vs 88%) — and frame the result as "broad improvement around the frozen base policy rather than uniform task-level dominance." They also state the structural ceiling explicitly: **absolute performance remains constrained by the frozen base's semantic action quality**, since ViTaR never generates a new action direction. And on the physical results: *"Given the three-task, 20-trial physical protocol, these results provide limited transfer evidence rather than resolving the sim-to-real gap."*

**vs. direct residual RL** (same frozen reference, residual coordinates, decision points, safety limits):

| Method | Recovery ↑ | Preservation ↑ | Unsafe ↓ | Avg |
|---|---|---|---|---|
| PPO-option | .55 | .66 | .25 | 36.0% |
| SAC-residual | .45 | .74 | .31 | 32.0% |
| **Full ViTaR** | **.79** | **.92** | **.04** | **61.3%** |

The unsafe rate is the striking column: **0.04 vs 0.25/0.31**. Effect-guided selection is not just more successful but 6–8× safer than RL under a matched bounded-residual protocol.

**The two tactile signals do different jobs** — the ablation that justifies the design:

| Variant | Recovery ↑ | Unsafe ↓ | Avg |
|---|---|---|---|
| w/o State (`m_i`) | .46 | .16 | 41.0% |
| w/o Tactile (`z_i`) | .61 | .13 | 46.0% |
| w/o Both | .40 | .19 | 36.0% |
| Fixed scale | .57 | .11 | 45.0% |
| **Full** | **.79** | **.04** | **61.3%** |

Removing `m_i` collapses **recovery** (0.79 → 0.46) — it governs *whether* to intervene. Removing `z_i` collapses **success** (61.3 → 46.0) with recovery largely intact — it governs *how much*. Fixing the gain costs 16.3 points. Complementary, non-redundant.

**EGM component ablation** (P>B / B>N are preference-ordering accuracies; FPR at 80% recall):

| Variant | P>B ↑ | B>N ↑ | FPR ↓ | Avg |
|---|---|---|---|---|
| RawScore (pointwise regression) | .63 | .60 | .40 | 38.0% |
| PairRank | .73 | .72 | .25 | 40.0% |
| + outcome-gap weighting | .71 | .76 | .29 | 44.0% |
| + restoration-aware weighting | **.83** | **.88** | **.12** | **61.3%** |

The last row is the big jump: down-weighting **unreliably restored branches** adds 17.3 points. Since the whole supervision scheme depends on comparing residuals from the *same* contact state, how faithfully that state was restored is the dominant source of label noise — a subtle and generalisable finding for any preference-based robot learning that resets to a decision point.

## 7. What it adds that the others don't

The **bounded-modulation paradigm** and the safety numbers that back it. It is the only tactile-VLA here that cannot forget its pretraining, cannot invent an action direction, and reports a 0.04 unsafe rate against 0.25–0.31 for RL alternatives. The clean separation of *whether to intervene* (marker descriptor) from *how much* (tactile summary), each independently ablated, is a design distinction nothing else in this survey makes. Contrast [[restacvla]] and [[omnitactune]], which also add residuals but let tactile shape them continuously.
