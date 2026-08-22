# ViTaL — Inference-time Policy Steering via Vision and Touch

**arXiv:2606.14981** · Carnegie Mellon University (Wu, Si, Kroemer, Temel, Bajcsy) · Jun 2026 · [site](https://yilin-wu98.github.io/vital_website/)

**One line.** Steers a **frozen** pre-trained diffusion policy at deployment by verifying candidate actions against predicted visual *and* tactile futures — with the first **language-conditioned tactile reward**, scored directly in latent space.

## 1. What "tactile" means here

**GelSight Mini** on a Robotiq gripper fingertip — dense contact geometry, deformation, and force-related signals. Observations are `o_t := [o^v_t, o^τ_t, q_t]` (RGB from third-person + wrist cameras, tactile images, proprioception = EE pose + gripper width).

The motivating example is precise: in **pipetting**, whether the robot is about to dispense mid-transport, or whether its grasp will produce the right force at the goal, **is not observable from wrist or third-person images at all**.

## 2. The two challenges it identifies

**(a) Modality-dependent observability across temporal scales.** Candidate action sequences must be rolled out *long enough* to reveal semantically distinct visual outcomes (which cup is the robot approaching?), whereas **tactile outcomes are local and transient** — contact events that get obscured inside long-horizon visual predictions (how hard is the dropper head being squeezed?).

**(b) No single unified reward exists.** The relative importance of visual and tactile outcomes **shifts across task phases**, so rewards must themselves be phase-conditioned.

## 3. Model — bi-level optimisation

The decomposition matches each modality with its strength:

**High level — visual mode selection.** N candidate action chunks `a_t ~ π_θ(·|o_t)` from the frozen diffusion policy are rolled out through a **visuo-tactile latent world model**; a visual verifier picks the sequence best satisfying the long-horizon visual objective. *What behaviour should the robot execute?*

**Low level — tactile-guided diffusion editing.** The selected sequence becomes an **action anchor**, refined over a shorter horizon by tactile-guided diffusion editing to satisfy local contact requirements while staying close to the visual plan. *How should that behaviour be physically realised?*

The world model operates in **latent space**, so actions can be scored **without decoding images or tactile frames** — which is what makes the steering affordable.

## 4. How tactile enters the model — a text-conditioned tactile reward

The novel piece. **Text is used as the shared task representation bridging visual and tactile reasoning**: a high-level instruction L ("transfer liquid to the blue cup") is decomposed into **phase-level visual and tactile objectives** (`ℓ^v`, `ℓ^τ`), each evaluated by a semantically aligned verifier.

- **Visual verifier** (ROBOMETER) scores decoded visual predictions for semantic progress.
- **Tactile verifier** scores **predicted tactile latents against textual contact objectives** — e.g. "Grasp Heavily" vs. "Grasp Lightly" — directly in the aligned latent space.

The claimed benefits are concrete: the verifier can distinguish contacts, adapt across task phases through language, and guide actions **without hand-designing task-specific tactile rewards or decoding tactile predictions**. Compared to TouchGuide, which learns action-space verifiers by contrastive learning, ViTaL needs no positive-sample demonstrations and does model dynamics.

## 5. Experiment setup

Three real-world contact-rich tasks: **pipette transfer** (multi-phase, multi-target-cup), **wiping** (orange marks), **peg insertion** (right hole). Success is decomposed into **Overall / Visual / Contact** components.

Baselines: base policy; **unimodal steering** (visual-only, tactile-only); **naive multimodal fusion** (normalise visual and tactile rewards separately across N = 10 samples and sum them for selection).

Reward quality is evaluated separately by **preference-order accuracy** over 40 trials: select rollout pairs with differing human preferences, rank each pair by summed predicted reward divided by rollout length, measure agreement with human ordering — on both ground-truth (GT) and world-model-predicted (Pred) futures.

## 6. Does it work?

**Main results:** +**51%** overall success over the base policy; ≥**33%** over unimodal steering; ≥**20%** over naive multimodal fusion.

The failure modes are diagnostic and cleanly split:
- **Visual-only steering** picks the right cup but produces imprecise contact — spilling.
- **Tactile-only steering** achieves adequate grasp force but **misses the intended visual goal**.
- **Naive fusion** suffers **reward imbalance** — the visual objective dominates selection, yielding actions that look globally plausible but never establish proper contact: marks partially wiped, peg misaligned, liquid spilled.

**Reward accuracy** (preference-order, %, ±binomial SE over 40 trials):

| Source | Visual: Wiping | Insertion | Pipette | Avg | Tactile: Wiping | Insertion | Pipette | Avg |
|---|---|---|---|---|---|---|---|---|
| GT | 80.0±6.3 | 70.0±7.2 | 100.0±0.0 | 83.3±5.9 | 85.0±5.6 | 70.0±7.2 | 80.0±6.3 | 78.3±6.5 |
| Pred | 82.5±6.0 | 72.5±7.1 | 100.0±0.0 | **85.0±5.6** | 90.0±4.7 | 77.5±6.6 | 77.5±6.6 | **81.7±6.1** |

Both verifiers clear 70% on every task. The interesting anomaly: **Pred rewards are sometimes *more* accurate than GT rewards**. The authors' explanation is credible — predicted latents **smooth observation noise** and have lower variation than real rollouts, making them easier to rank. A world model that is slightly wrong but consistently wrong can be a better ranking substrate than reality.

Qualitatively, the visual verifier fires on target-cup identity and phase transitions; the tactile verifier tracks the gentle→firm grasp transition, **consistent with marker-tracking force estimates** — an independent check that the language-conditioned latent score corresponds to physical force.

**Cost:** ViTaL adds **0.05 s per policy inference step** (8 actions) over naive fusion. Bi-level is cheaper *and* better than fusing rewards, because visual lookahead runs on few long rollouts and tactile refinement is a short-horizon edit.

**Stated limitations.** Dependence on latent world-model fidelity, with compounding prediction errors affecting verification of subtle contact events; and — notably honest — the tactile verifier is *"limited by tactile encoders pretrained on much smaller datasets than modern VLMs"*, pointing at the cross-sensor foundation-model line ([[ftp-1]], [[anytouch2]], [[htt]]) as the bottleneck on its own contribution.

## 7. What it adds that the others don't

**Language as the bridge between visual and tactile verification**, and the resulting text-conditioned tactile reward — the only mechanism in this survey that lets a phase-level instruction ("grasp lightly") score a *predicted* contact state without a task-specific learned reward. The bi-level decomposition is also the cleanest answer to the temporal-scale mismatch that [[touchworld]] solves with a control hierarchy and [[vt-wm]] solves with different context lengths: here it becomes long-horizon selection followed by short-horizon editing, on a frozen base policy that is never retrained.
