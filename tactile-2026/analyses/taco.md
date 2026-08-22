# TACO — Tactile World Model as a Self-Corrector for Scalable VLA Post-Training

**arXiv:2607.02840** · Peking University + AI² Robotics + Sun Yat-sen + Beihang (S. Liu, Jia, Yan, J. Liu, X. Zhang, Feng, Guo, S. Zhou, Shi, S. Zhang) · Jul 2026 · [site](https://taco-wm.github.io/)

**One line.** Uses a tactile world model to *manufacture the corrective demonstrations humans would otherwise have to teleoperate*, on the observation that contact failures are localised, not semantic — the policy knows what to do, it just can't recover.

## 1. What "tactile" means here

**6-DoF force–torque per arm**, concatenated into a **12-dimensional** vector `F_t ∈ R^12` (left + right). Xense tactile sensors on the hardware.

The motivating observation is precise and worth keeping: near contact transitions, **visual observations change only slightly while tactile signals shift significantly** — slippage, insufficient pressure, abnormal torque. Their examples: in *Wipe Whiteboard* the eraser covers the mark without enough force to remove it; in *Twist Bottle Cap* the gripper aligns with the cap but generates no effective twisting torque. Both look like success on camera.

Notably the paper also cites prior evidence that **naively adding tactile inputs can impair pre-contact perception and grounding** — which is the problem the knowledge-insulation design exists to solve.

## 2. Data curriculum — a self-generating loop

The curriculum is the method. **Recognize → Imagine → Label**, iterated:

1. **Recognize** — a unified progress-action model estimates task progress from visual *and* tactile signals, identifying **failure-adjacent states** in real rollouts.
2. **Imagine** — the visuo-tactile generation model produces a local correction segment from that state, jointly denoising future video and force.
3. **Label** — the *same* progress-action model labels imagined segments with executable corrective actions.

Generation-model training itself is two-stage: fine-tune **Wan2.2-TI2V-5B** on broad robot trajectories for visual fidelity and robot-scene consistency, then adapt to contact-rich demonstrations with sliding windows.

**Real-to-imagined ratio is a scaling knob** — see results.

## 3. Model

**Visuo-tactile generation model** — Wan2.2-TI2V-5B DiT backbone. Video latents `X^v ∈ R^{B×N_v×d}`, force sequence `F ∈ R^{B×T×12}` tokenised as `X^f = T_η(F) ∈ R^{B×T×d}`.

**Unified progress-action model** — `(â_t, p̂_t) = U_φ(I_t, F_t)` with `â_t ∈ R^7`, `p̂_t ∈ [0,1]`. A **DINOv2** visual pathway with a direction-aware decoder for spatial grounding, plus an **MLP tactile pathway** for normalised 12-D force-torque. The fused embedding `[z^v_t ; z^f_t]` feeds an action head and a sigmoid progress head.

## 4. How tactile enters the model

Three mechanisms, and the second is the most transferable idea in the paper.

**(a) Concatenation into DiT self-attention.** `X = [X^v ; X^f] ∈ R^{B×(N_v+T)×d}` — full bidirectional video–force interaction, no gating. Joint flow-matching loss:
```
L_joint = ‖u^v_ψ − (ξ^v_1 − ξ^v_0)‖² + λ_f ‖u^f_ψ − (ξ^f_1 − ξ^f_0)‖²
```
with video and force sharing the same sampled denoising timestep.

**(b) Temporal RoPE alignment.** Wan2.2 applies RoPE over a **3D video latent grid**, but force tokens carry only a temporal axis. So each force token is mapped onto the video temporal axis:
```
ρ(i) = round( i/(T−1) · (f−1) ),   i = 0 … T−1
```
using temporal RoPE at ρ(i) with **spatial RoPE fixed at 1 + 0j**. Plus **first-frame force anchoring** — `F_0 ∈ R^12` is kept clean to reduce contact-state ambiguity. This is the cleanest published solution to a problem every video-backbone tactile model faces: how do you position-encode a 1-D physical signal inside a 3-D visual grid?

**(c) Knowledge-insulated tactile adaptation.** Stop-gradient isolates the pretrained VLM backbone; tactile learning is routed **only to the action expert**. Combined with **advantage-conditioned training** (binary labels: corrective segments = 1, failures = 0) as an offline RL objective, so failures are used rather than filtered away.

## 5. Experiment setup

Six real-world contact-rich tasks — including Move Hanoi Rings, Twist Bottle Cap, Wipe Whiteboard, Insert Flower. Two post-training iterations. Baselines: base policy, **Filtered BC** (retrain on filtered successful rollouts), and TACO without knowledge insulation. Generalisation tested on unseen backgrounds, objects, and positions.

## 6. Does it work?

**Main result.** After two iterations, TACO improves average success by **+44%** over the base policy, **+39%** over Filtered BC, and **+32%** over the same pipeline without knowledge-insulated tactile adaptation. It also reaches fewer average completion steps — smoother execution with fewer pauses and indecisive contact transitions.

The Filtered BC comparison is the important one. Filtered BC saturates because its successful rollouts **contain no recovery behaviour at failure-adjacent states** — it reinforces the narrow demonstration manifold. The action-distribution analysis confirms this directly: projecting end-effector x-y poses over 40 successful rollouts on Insert Flower, the base policy is narrowly concentrated around the demonstration manifold, Filtered BC stays there, and **TACO progressively broadens across iterations**.

**Ablations on where tactile matters:**

| Setting | Force val. loss ↓ | Action val. loss ↓ | VOC ↑ | Failure-loc. acc ↑ | **Real SR ↑** |
|---|---|---|---|---|---|
| w/o tactile **generation** (V→V) | 0.025 | 0.78 | 0.87 | 0.28 | **0.28** |
| w/o tactile **labeling** (V+F gen, V label) | 0.038 | 0.88 | 0.90 | 0.65 | **0.65** |
| **TACO** (V+F throughout) | **0.019** | **0.94** | **0.95** | **0.82** | **0.82** |

This is a genuinely informative decomposition. Removing tactile from *imagination* collapses success to 28% — visual imagination alone cannot capture contact-state transitions. Removing tactile only from *labeling* still costs 17 points (0.82 → 0.65), showing touch is **most effective when it directly participates in corrective action and progress prediction**, not merely as an imagined observation. Failure-localisation accuracy tracks success almost exactly, which suggests the whole pipeline's performance is gated by whether you can *find* the failure-adjacent state at all — and that is a tactile problem.

**Scaling imagined data** (Insert Flower):

| Real : imagined | Success |
|---|---|
| base policy | — |
| 1:2 | 70% |
| 1:4 | 93% |
| 1:8 | **97%** |

Monotonic, and still improving at 1:8. The authors read the 1:4 → 1:8 gain as broader coverage of failure-adjacent contact states.

**Generalisation.** One TACO iteration using **OOD imagined corrections** raises success under unseen backgrounds, objects and positions, where the base policy degrades on all three. The authors note these OOD settings lie outside the world model's training distribution in vision, tactile *and* action — yet it still generates effective corrections.

**Stated limitation.** Corrections are generated **offline**, not online during deployment; tighter world-model/policy coupling and online correction generation are future work.

## 7. What it adds that the others don't

It answers a question no other tactile world model asks: *what is imagination actually for?* Not planning ([[vt-wm]]), not action generation ([[n0-twam]]), not generic data augmentation ([[vitacworld]]) — but specifically **synthesising the recovery behaviour that expert demonstrations structurally cannot contain**. The temporal-RoPE alignment of a 1-D force sequence into a 3-D video latent grid, and the stop-gradient insulation of the pretrained VLM from tactile losses, are both reusable beyond this paper.
