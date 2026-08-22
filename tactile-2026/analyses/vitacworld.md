# ViTacWorld — Scaling Visuo-Tactile World Models for Contact-Rich Robot Manipulation

**arXiv:2607.22530** · ShanghaiTech University + InstAdapt (Y. Huang, Sang, Lu, Ni, Wu, Guo, Shi, J. Wang) · Jul 2026 · [site](https://vitacworld.github.io/)

**One line.** Uses the world model as a **data generator**, not a planner — synthesising dream visuo-tactile rollouts to train downstream tactile policies, which lifts a tactile π₀.₅ from 42.5% to 67.5%.

## 1. What "tactile" means here

Tactile is modelled as **an additional camera view**. The observation is `o_t = {I_t^v}` over `V = {main, wrist, tactile}` — external camera, wrist camera, and an **image-like tactile stream**, all encoded by the same VAE.

That framing is the paper's central simplification, and it comes with an explicit rationale: tactile signals are *directly grounded in local contact geometry and force response*, so simulated tactile may have a **smaller sim-to-real gap than simulated vision**. If touch is easier to simulate faithfully than pixels, simulation is a better scaling substrate for touch than for sight — an argument no other work in this survey makes this directly.

A view-presence mask `m ∈ {0,1}^{|V|}` handles heterogeneous training data where not every source has every view.

## 2. Data curriculum — three sources, three stages

The curriculum *is* the contribution:

**Pretraining**
1. **Public real visuo-tactile datasets** — broad coverage, wrong domain
2. **Task-aligned simulation data** — built via real-to-sim for the target scenes/tasks, complementing the public data

**Finetuning**
3. **Real task demonstrations** — 300 expert demos: 120 charger plugging, 50 cucumber peeling, 62 U-block insertion, 68 cuboid insertion, collected by FACTR teleoperation
4. **Real policy rollouts** — 50 per task, **including both successes and failures**, used to tune the world model to the state distribution the downstream policy actually induces

Then the generation loop: the warmed-up tactile π₀.₅ acts as rollout policy, candidate trajectories are generated, filtered by task success and visual-tactile plausibility, and **200 high-quality successful rollouts** are merged with the expert demonstrations.

Hardware: Intel RealSense D435 third-person + ZED Mini wrist.

## 3. Model

A pretrained **action-conditioned robot video world model** (Wan/Cosmos-lineage DiT) extended to visuo-tactile generation while **preserving its backbone and action-conditioning pathway**. The stated design principle — inherit motion priors, spend capacity only on tactile dynamics — is why adaptation is cheap.

Prediction target: `ô_{t+1:t+H} = f_θ(o_t, u_{t:t+H−1}, m)`, where each action `u` is relative end-effector motion plus a gripper command.

## 4. How tactile enters the model

Two mechanisms, both about **keeping the streams separate before mixing them**:

1. **Stream-identity embeddings** `e_v` for visual and tactile streams, projected through a **zero-initialised** projection `P` into the same **AdaLN** modulation path as timestep and action conditioning:
   `Z̃_b^v = SelfAttn_v( AdaLN(Z_{b−1}^v ; c_b + P(e_v)) )`
   The zero-init means the model starts as the original video world model and *learns* to differentiate views.

2. **Explicit cross-view attention** after in-view self-attention:
   `Z_b^v = CrossViewAttn_v( Z̃_b^v, {Z̃_b^{v′}}_{v′≠v} )`

The stated purpose is to **"avoid uncontrolled mixing between camera and tactile tokens during ordinary self-attention"** — a middle position between the free shared attention of [[n0-twam]] and the explicit gating of [[dream-tac]]: in-view attention first, then a dedicated cross-view channel.

## 5. Experiment setup

Four contact-rich tasks: **Charger Plugging** (hardest — small pose errors fail insertion even when the visual trajectory looks right), **Cucumber Peeling**, **U-Block Insertion**, **Cuboid Insertion**.

Three downstream policies: vision-only π₀.₅, tactile π₀.₅, ACT + tactile. 10 real-robot trials per task per policy.

Two roles evaluated: **policy improvement** (augment training data) and **policy evaluation** (predict action-conditioned visuo-tactile outcomes under controlled action sequences).

## 6. Does it work?

**Policy improvement** (real-robot success, average over four tasks):

| Data source | ACT + tactile | π₀.₅ | π₀.₅ + tactile |
|---|---|---|---|
| Expert only | 15.0 | 35.0 | 42.5 |
| **+ ViTacWorld rollouts** | **27.5** | **47.5** | **67.5** |

Gains across all three, including the **vision-only** policy (+12.5) — which the authors correctly read as evidence the rollouts provide genuine supervision rather than overfitting to the generator's own policy. The largest gain goes to the tactile policy (+25.0), consistent with the generated *tactile* observations carrying real contact information.

The behavioural analysis is more convincing than the table. With expert data only, failures happen **early** — inaccurate grasp position, unstable pick, poor approach alignment. After augmentation, policies show better grasp localisation and robustness to initial pose variation, and for tactile policies specifically, during the contact phase the policy **performs small corrective motions and re-aligns using contact feedback** rather than failing after one misaligned insertion attempt. So the dream data improves both pre-contact generalisation and in-contact correction.

**Generation quality** (held-out real validation clips, given first frame + ground-truth actions):

| View | Variant | PSNR ↑ | SSIM ↑ | LPIPS ↓ |
|---|---|---|---|---|
| Main | w/o pretraining | 22.718 | 0.7859 | 0.0781 |
| | w/o task-aligned sim | 23.128 | 0.8011 | 0.0687 |
| | **Full** | **24.258** | **0.8286** | **0.0513** |
| Wrist | w/o pretraining | 21.080 | 0.6649 | 0.1084 |
| | **Full** | **21.925** | **0.6962** | **0.0725** |
| Tactile | w/o pretraining | 34.967 | 0.9204 | 0.0211 |
| | **Full** | **35.225** | **0.9318** | **0.0157** |

The tactile row carries a useful methodological warning the authors flag themselves: **tactile PSNR/SSIM barely move (34.97 → 35.23)** because most tactile frames are non-contact or near-static, so pixel-level similarity is already high. Only **LPIPS** (0.0211 → 0.0157, −26%) reflects the improvement in generated contact patterns. Any paper reporting tactile-prediction PSNR without this caveat is reporting mostly the empty frames.

Both ablation rows matter: pretraining alone helps substantially over direct real tuning, and **task-aligned simulation adds a further increment on top** — supporting the sim-is-good-for-touch hypothesis.

**Stated limitation.** Selecting successful dream data still relies **partly on manual inspection**; automated filtering with VLMs or multimodal evaluators is future work. Given the whole pipeline's value rests on that filter, this is a real bottleneck, not a formality.

## 7. What it adds that the others don't

The only tactile world model positioned as a **reusable data generator** rather than a planner or a policy backbone — its output is a dataset that improves *other people's* policies, including vision-only ones. The claim that tactile has a smaller sim-to-real gap than vision, and the finding that task-aligned sim adds on top of large-scale real pretraining, are the sharpest arguments in this survey for [[univtac]]-style simulation platforms. Contrast [[deform360]], which uses a real capture rig for the same scaling problem, and [[n0-twam]], which just collects 30,000 hours.
