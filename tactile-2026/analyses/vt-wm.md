# VT-WM — Visuo-Tactile World Models

**arXiv:2602.06001** · University of Washington + **FAIR, Meta** (Higuera, Arnaud, Boots, Mukadam, Hogan, Meier) · Feb 2026

**One line.** The paper that named the failure mode the rest of 2026's world-model papers are trying to fix: vision-only world models *hallucinate* — objects disappear, teleport, or move without being touched — and touch is what stops it.

## 1. What "tactile" means here

**Digit 360** sensors as fingertips on an **Allegro Hand** on a Franka Panda. Tactile is a stream of images of elastomer deformation at **30 fps**, capturing contact area, force, shape and texture.

The paper's argument for *why* touch and not more cameras is the sharpest statement of the case in the literature: from an exocentric camera, a hand resting on a cloth and a hand pressing a cloth look identical, but only one of them will move the cloth when the hand translates. Vision gives global context about kinematics and scene; it does not reveal **the state of physical contact**. Two identical video frames of a hand around a cup lead to different futures — lifted or not — and only tactile disambiguates.

## 2. Data curriculum

Multi-task contact-rich manipulation: **place fruits, push fruits, wipe with cloth, stack cubes, scribble with marker**, plus a held-out novel task (**place plate in dish rack**) used for the low-data study with only **20 demonstrations**.

The temporal design is the interesting part, because the two modalities are deliberately given **different horizons**:

| Stream | Rate | Context |
|---|---|---|
| Exocentric video | 6 fps, 320×192 | 9 frames = **1.5 s** |
| Tactile (×4 Digit 360) | 30 fps | **2 frames = 0.16 s** per sensor |
| Actions | 30 Hz, chunked ×5 | 6 Hz effective, 9 steps |

The stated rationale: contact information is higher-frequency and local, so it needs a short horizon; vision supplies the slower global context. Max context length 9 frames for both.

## 3. Model

A **latent-state, action-conditioned world model** — nothing is predicted in pixel space.

- **Vision encoder** — frozen **Cosmos tokenizer**, framewise → latents `s_k`
- **Tactile encoder** — frozen **Sparsh-X** (the authors' own tactile foundation model) → latents `t_k`
- **Predictor** — a **12-layer transformer** estimating `(s_{k+1}, t_{k+1}) ~ P_φ(s_k, t_k | a_k)`

Actions are proprioceptive deltas (translation, quaternion rotation) plus a binary open/close hand state.

## 4. How tactile enters the model

Cleanly and symmetrically, which is what makes this a reference design:

1. Both modalities encoded by **pretrained frozen** encoders, augmented with sinusoidal positional embeddings and projected into a unified `R^{(b,t,s,d)}`.
2. Vision and tactile tokens **concatenated along the spatial dimension** into one sequence.
3. **Factorised spatio-temporal self-attention** — spatial attention lets all tokens within a timestep interact; temporal attention tracks each token across past timesteps. This avoids the O((THW)²) cost of full spatiotemporal attention.
4. **Action conditioning by cross-attention** after each self-attention block — vision-touch tokens cross-attend to action tokens, alternating self/cross so latents are iteratively refined by both observation and control.
5. **RoPE** throughout; modality-specific output heads project back.

Training combines **teacher forcing** (L1 on next-step latents for both modalities) with **autoregressive sampling loss** to keep long-horizon rollouts coherent.

Note what is *not* here: no contact gate, no attention bias, no tactile-specific fusion trick. Touch is just another token stream with a shorter context. Several later works ([[dream-tac]], [[tactile-wam]]) argue that symmetric treatment is exactly the mistake.

## 5. Experiment setup

Two evaluations of *imagination quality* that are novel and worth adopting:

- **Object permanence** — normalised Fréchet distance between the ground-truth and imagined trajectory of a manipulated object, tracked with CoTracker.
- **Causal compliance** — the same metric applied to keypoints on objects that receive **no external force and should not move**. A high distance means the model hallucinated motion, i.e. violated Newton's first law.

Then planning: **CEM** over a goal-conditioned energy — cost is latent distance between the final predicted visual state `s_{k+H}` and a goal image latent. Search space is `R^7` (3D translation + 3D orientation + binary hand state). Open-loop, **zero-shot** transfer to the real robot, five trials per task from distinct initial conditions.

## 6. Does tactile actually help?

**Imagination.** VT-WM reduces normalised Fréchet distance by **~33%** on object permanence and **~29%** on causal compliance, averaged across tasks. The paper reports paired t-tests rather than just means, which is unusual and welcome — causal compliance is significant on place fruits (t = 3.66, p < 0.001), push fruits (t = 2.28, p < 0.05) and wipe with cloth (t = 2.99, p < 0.01), but **not** on stack cubes (p = 0.09), and **VT-WM is worse on scribble with marker** (t = −1.22, p = 0.23). Relative reductions: 43.6% / 16.4% / 66.1% on the three significant tasks.

The qualitative case is the clearest illustration in the survey: the robot wipes *just above* a cloth without contact. Ground truth — cloth keypoints stationary. V-WM — significant keypoint displacement and cloth deformation, conditioned on the true actions. VT-WM — few artifacts. The vision-only model cannot tell contact from near-contact, so it invents the physics.

**Zero-shot real-robot planning** (success rate, 5 trials each):

| Task | Gain from tactile |
|---|---|
| Reach button | 0 (both 100%) |
| Push fruits | **+10%** |
| Reach & push | **+35%** |
| Wipe cloth | **+31%** |
| Stack cubes | **+11%** |

The pattern is exactly what the hypothesis predicts: free-space kinematic tasks are already solved by vision (matching V-JEPA-2AC), and gains concentrate in **multi-step tasks requiring sustained contact**.

**Low-data adaptation.** Fine-tuning the multi-task VT-WM on **20 demonstrations** of a novel task (place plate in dish rack) gives **77%** planning success over ten trials with randomised initial plate-in-grasp pose. The dominant failure is placing the plate *beside* the rack — a precision error, not a task-understanding error.

## 7. What it adds that the others don't

Two things the field then built on. First, the **diagnostic framing**: object permanence and causal compliance as measurable properties of imagination, with statistical tests, rather than downstream success as the only proxy. Second, the demonstration that **frozen pretrained encoders on both sides** (Cosmos + Sparsh-X) plus a plain factorised transformer is sufficient — establishing the baseline that [[n0-twam]] scales, [[dream-tac]] gates, and [[tactile-wam]] makes asymmetric.
