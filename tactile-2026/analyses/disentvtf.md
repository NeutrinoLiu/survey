# Disentangling Visuo-Tactile Foresight — Oracle-Guided Interface Discovery for World Action Models

**arXiv:2608.00547** · Brigham Young University + Beijing Academy of Science and Technology (Yao, Ding, Yu) · Aug 2026

**One line.** Removes the learned predictor entirely and hands the policy a **verified successful future**, so that the question "how should predicted touch be delivered to the action expert?" can be answered without confounding it with "was the prediction any good?"

## 1. What "tactile" means here

Left and right **tactile RGB** images alongside head/wrist RGB and an 8-D joint-position vector. But the paper is not about tactile representation — it is about the **future-to-action interface**, and touch is the modality where that interface matters.

## 2. The methodological problem, stated precisely

End-to-end evaluation of a tactile world action model entangles **four** distinct failure sources:

1. **Physically invalid visual futures** — a visually convincing continuation that violates contact mechanics or is not executable
2. **Insufficient trust in the future branch** — unstable predictions during training teach the action branch to ignore future information altogether
3. **Incorrect or cross-modally mismatched tactile futures** — matching tactile evolution to a predicted video is especially hard
4. **A poor future-to-action interface** — even a *correct* future encoded in a form the action branch cannot read

A single task score cannot separate these. So every architecture comparison in the world-model cluster ([[n0-twam]] vs [[tactile-wam]] vs [[feelworld]] vs [[hitac-wam]]) is measuring some mixture of all four.

## 3. Method — Oracle Visuo-Tactile Foresight (OVTF)

**Fix the future provider.** Instead of a learned predictor, supply paired RGB and tactile futures drawn from **successful demonstration trajectories that are chronologically ordered, matched to the same initialisation, and verified to succeed in simulation**. Sources 1–3 are eliminated by construction; only the interface remains.

The oracle is deliberately **not** a contiguous t+1:t+H prediction. For an action-window reference index `τ_ref`, up to **N = 100** frames are uniformly sampled from `τ_ref` to the trajectory end, each carrying a **normalised remaining-horizon phase**:

```
φ_i = (τ_i − τ_ref) / max(1, T − 1 − τ_ref) ,   φ_i ∈ [0,1]
```

At rollout, `τ_ref = min(⌊p/2⌋, T−1)` for policy-call index p.

## 4. How tactile enters — AFM vs IFM

Twelve future-memory slots (**4 head-vision · 4 left-tactile · 4 right-tactile**), each a *phase-local* token that aggregates evidence from a local trajectory neighbourhood via a soft Gaussian phase prior added to the attention score:

```
score^h_{k,i} = (q^h_k)ᵀ k^h_i / √d  −  (φ_i − c_k)² / (2 × 0.18²)
```

Note it is a **soft prior, not a hard temporal window** — content attention can still select among all valid source positions.

**AFM (Asymmetric Phase-Local Future Memory)** routing:
```
H ← H,    L ← (L, H),    R ← (R, H),    L ↔ R prohibited
```
Each tactile slot performs **a single joint attention operation** over the concatenation of its own tactile sequence and the visual sequence — not tactile first, then vision.

**IFM (Modality-Isolated Future Memory)** — identical in every respect (slots, phase anchors, parameterisation, future source, data, optimisation budget) except the `H → L` and `H → R` edges are removed:
```
H ← H,    L ← L,    R ← R
```

Current observations go through **frozen** UniVTAC-ACT* ResNet-18 encoders; futures go through **trainable** ResNet-18 encoders and AFM before entering the same ACT policy (4-layer encoder, 7-layer decoder, 50 action queries → 50×8 action chunk).

## 5. Experiment setup

Seven UniVTAC simulation tasks. Four conditions:

- **AFM Oracle** — full method with oracle futures
- **IFM** — modality-isolated, same oracle
- **AFM Forced-Zero** — oracle-trained policy, future inputs **disabled at evaluation**
- **AFM Zero-Train** — trained without futures at all
- Baseline: **UniVTAC-ACT\***

## 6. Findings

| Condition | Average success |
|---|---|
| UniVTAC-ACT* baseline | 14.9% |
| AFM Zero-Train | 20.0% |
| AFM Forced-Zero (oracle-trained, futures off at test) | 19.4% |
| IFM (oracle) | 23.7% |
| **AFM (oracle)** | **32.0%** |

Three conclusions the authors extract, each of which an end-to-end score could not:

**(a) Interface routing matters, by 8.3 points.** AFM vs IFM under an identical oracle, memory budget, data, and optimisation. Crucially, in *both* conditions the modality tokens **can re-attend to each other inside the subsequent ACT encoder** — so downstream self-attention has the opportunity to recover cross-modal information and does not. Explicit asymmetric fusion **at the actor-facing interface** contributes organisation that later self-attention cannot reconstruct.

**(b) The policy genuinely uses test-time future content.** Oracle-trained AFM falls from 32.0% to **19.4%** under Forced-Zero. It is not merely benefiting from extra parameters.

**(c) The architecture is good independent of futures.** AFM Zero-Train reaches 20.0%, above the 14.9% baseline — so the AFM-compatible policy path is itself a stronger design.

The honest reading of the absolute numbers is sobering: **even with a perfect, verified, physically executable future, average success is 32.0%** on seven UniVTAC tasks. That is an upper bound on what any amount of improvement to tactile *prediction* can buy in this setup, and it sits well below what the learned end-to-end systems report on their own benchmarks — a reminder that cross-paper comparison of success rates is largely meaningless.

The direction of the asymmetry is also notable: AFM lets **vision inform tactile** memory, while [[tactile-wam]], [[hitac-wam]] and [[feelworld]] all restrict the *reverse* direction (tactile informing vision). Both asymmetries are defensible and they are not in conflict — vision→tactile is contextualisation, tactile→vision is pollution.

## 7. What it adds that the others don't

The **experimental design**. It is the only work in this survey that isolates one link in the world-model chain and holds everything else fixed, and the only one to establish that the future-to-action *interface* is a distinct bottleneck from future *accuracy*. The blocked-cross-tactile, phase-aligned routing result — that explicit organisation before the policy beats letting a transformer sort it out — is a concrete finding, and the 32% oracle ceiling is a useful piece of calibration for the whole cluster.
