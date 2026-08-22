# HapticVLA — Contact-Rich Manipulation via VLA Model without Inference-Time Tactile Sensing

**arXiv:2603.15257** (v2, Aug 2026) · Intelligent Space Robotics Lab, Skoltech (Gubernatorov, Sannikov, Mikhalchuk, Kuznetsov, Artemov, Oluwatobi, Fernando, Asanov, Guo, Tsetserukou) · Mar 2026

**One line.** Distils tactile awareness into a **compact token predicted from vision and state**, so the deployed robot needs **no tactile sensor at all** — and beats VLAs that *do* have tactile at inference.

## 1. What "tactile" means here — a reward, then a token

Two transformations, in sequence.

**Stage 1: tactile → safety reward.** Rewards are computed **offline** per episode from manipulator state and tactile feedback, penalising **excessive grasping force on fragile objects** and suboptimal grasping trajectories. Tactile is thus never an input; it is a source of supervision.

**Stage 2: reward-trained expert → distilled tactile token.** A compact token summarising tactile-aware behaviour, which a student learns to predict from vision and state alone.

The critique motivating this is worth noting: existing methods *"encode raw sensor signals as a vision modality, overlooking the unique modality of tactile sensing: unlike scene vision, which captures remote photometric properties, tactile sensing captures local mechanical interactions."*

The practical motivation is different from everyone else's in this survey and is about **reproducibility and cost**: high-end visuo-tactile sensors are expensive and often incompatible with common grippers, which limits reproducibility across platforms.

## 2. Model — three stages

1. **Offline Tactile Reward Calculation** — per-episode rewards from state + tactile, penalising high grasp force on fragile objects.
2. **SA-RWFM** (Safety-Aware Reward-Weighted Flow Matching) — fine-tune **SmolVLA** with a flow-matching action expert that incorporates the precomputed safety-aware tactile rewards, aligning tactile with state and vision.
3. **Tactile Distillation (TD)** — distil a compact tactile token from the SA-RWFM teacher; train a **conventional SmolVLA student** to predict that token from vision and state, giving tactile-aware action generation with **no on-board tactile sensor at deployment**.

## 3. How tactile enters the model

At training time, as a **reward weight** on flow matching — a different mechanism from every other VLA here, and closest in spirit to [[tactidex]]'s tri-component tactile reward, though used to weight a generative objective rather than to drive RL.

At inference time, **not at all**. The student predicts the tactile token from `(vision, state)`.

## 4. Experiment setup

Real-world contact-rich tasks, **20 evaluations per task**. Baselines: **SmolVLA**, **SmolVLA + SA-RWFM** (the teacher, with tactile at inference), **X-VLA**, **VLA-0**. A digital twin of the setup, code, models and datasets are released.

## 5. Does it work?

**Mean success 86.7%**, consistently outperforming baseline VLAs — *"including versions provided with direct tactile feedback at inference."*

That last clause is the result worth taking seriously and worth being sceptical about in equal measure. Beating the teacher, which has the sensor, is not what distillation usually does. Two readings are available, and the paper does not fully separate them:

- **The optimistic reading**: the distilled token is a *denoised, task-relevant summary* of tactile, while raw tactile at inference is noisy and partly irrelevant — so the student gets the signal without the noise. This is consistent with [[vital]]'s observation that predicted latents can be *better ranking substrates* than real observations because they smooth noise, and with [[at-vla]]'s finding that higher-dimensional tactile perturbs pretrained representations more.
- **The sceptical reading**: the tasks may be predictable enough from vision and state that a learned prior beats a sparse, mostly-silent sensor. [[unitacvla]] and [[at-vla]] both report sensor-free variants performing near their sensored ones, which suggests this regime is common — and that raises a question about how much *deployed* tactile is contributing across this whole literature.

The paper situates itself in a small but growing line — Gano et al. (low-fidelity tactile pretraining then disabled at inference), Beyond Sight, and **FD-VLA** (force distilled into a learnable token) — all of which report the same phenomenon.

## 6. What it adds that the others don't

It takes the "tactile as training signal" position to its conclusion: **if touch's value can be baked in offline, the sensor is a training-time cost, not a deployment requirement.** That reframes tactile hardware economics entirely — you instrument one data-collection rig, not every deployed robot. The safety-specific reward (penalising excessive force on fragile objects) is also a rare case of tactile being used for *harm avoidance* rather than task success, which is the axis [[softvtbench]] argues completion-only benchmarks systematically miss.
