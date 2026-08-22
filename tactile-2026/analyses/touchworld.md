# TouchWorld — A Predictive and Reactive Tactile Foundation Model for Dexterous Manipulation

**arXiv:2607.07287** (v2) · Harbin Institute of Technology, Shenzhen + PHANES AI (Zhou, Hong, Y. Li, Zhao, Cen, … Yang) · Jul 2026 · [site](https://phanes-lab.github.io/TouchWorld-website/)

**One line.** Diagnoses monolithic tactile VLAs as a **timescale** problem — slow semantic reasoning, medium action chunking and fast contact correction forced into one loop — and splits them into a 1 Hz / 10 Hz / 30 Hz hierarchy.

## 1. What "tactile" means here

Touch plays **two roles at two rates**, which is the paper's organising idea:

- **Predictive** — a *tactile subgoal*: what contact *should* look like when this subtask completes. Represented as a pressure map / tactile image.
- **Reactive** — a high-frequency tactile history feeding residual correction inside the robot control loop.

In the nominal policy, raw tactile readings are **rendered or normalised into tactile images**, so the VLA backbone receives touch in the same representational form as vision. The stated reason is compatibility: it keeps the policy aligned with image-language pretraining and **avoids introducing modality-specific tactile encoders into the VLA branch**. The refinement branch, by contrast, consumes raw tactile histories directly.

Pretraining data spans **rich human full-hand tactile data**, bimanual tactile data, and robot dexterous tactile data.

## 2. Data curriculum

Not the paper's focus, but three sources feed pretraining (human full-hand → bimanual → robot dexterous). The benchmark itself is six real-robot tasks, each with a clean and a **human-perturbation** variant: Water Flower, Tabletop Clearing, Cup Insertion, Power Plug Insertion, Pot Wiping, Tissue Pulling — covering long-horizon planning, precision insertion, continuous contact regulation, and soft-object handling.

## 3. Model — a three-rate hierarchy

| Layer | Rate | Component | Output |
|---|---|---|---|
| High-Level Planning | **1 Hz** | Subtask Planner + Tactile World Model | executable subtask `ℓ^sub_t`, visual-tactile subgoal `g_t` |
| Visuo-Tactile Goal-Conditioned Policy | **10 Hz** | diffusion Transformer, flow matching | nominal action chunk `Â_{t:t+H−1}`, context token `c_t` |
| Reactive Tactile Refinement | **30 Hz** | Tactile Residual Transformer | corrected window `Ã_{τ:τ+W−1}` |

Formally:
```
ℓ^sub_t = π_subtask(ℓ, I_t, m_t)                          # 1 Hz
g_t     = π_world(ℓ, ℓ^sub_t, I_t, X_t)                   # 1 Hz, refreshed only on subtask change
Â, c_t  = π_goal(ℓ, ℓ^sub_t, g_t, I_t, s_t, X_t)          # 10 Hz
Ã       = π_tactile(Â_{τ:τ+W−1}, s_{τ−k:τ}, X_{τ−k:τ}, c_t)  # 30 Hz
```

Memory `m_t` stores previous subtasks, predicted subgoals and execution status, letting the planner reason over progress from only the current observation.

A nice efficiency detail: the Tactile World Model **only re-predicts when the Subtask Planner detects a new subtask or meaningful state change** — stable phases reuse the previous tactile subgoal instead of regenerating futures every step.

The prompt to the action policy concatenates both levels: `"Task: ℓ ⊕ Current subtask: ℓ^sub_t"`, preserving long-horizon context while keeping the immediate target explicit. Free-form planner reasoning `r_t` is deliberately **kept outside the action interface**.

## 4. How tactile enters the model

Three distinct injection points, at three rates:

1. **Into the world model** — current tactile `X_t` conditions subgoal prediction.
2. **Into the nominal VLA** — as *tactile images* alongside RGB through the vision-language branch, plus optional predicted subgoal images / tactile subgoal latents appended as goal context.
3. **Into the refinement policy** — raw tactile history `X_{τ−k:τ}` plus proprioception `s_{τ−k:τ}`, producing residual corrections on a sliding lookahead window of length W.

Point 2 is the notable design choice: no tactile encoder at all in the VLA branch, on the argument that image-form touch inherits the pretrained visual pathway for free. Compare [[n0-twam]] (a dedicated tactile expert with private weights) and [[ftp-1]] (morphology-aware tactile tokens) — TouchWorld takes the opposite view, that the cheapest interface is the pretrained one.

## 5. Experiment setup

Six tasks × {clean, human perturbation} on a real robot. Baselines: **π₀.₅, FTP-1, GR00T N1.7** — notably including [[ftp-1]], a tactile foundation policy, rather than only vision baselines.

## 6. Does it work?

**Main results** (six-task average success %):

| Method | Clean | Human perturbation |
|---|---|---|
| π₀.₅ | 40.7 | 27.7 |
| FTP-1 | 49.3 | 35.2 |
| GR00T N1.7 | 39.3 | 26.0 |
| **TouchWorld** | **65.0** | **53.7** |

+15.7 clean, **+18.5 under perturbation** over the strongest baseline. The gap *widens* under perturbation, which is what the reactive-layer hypothesis predicts.

**Ablations** (four switches, six-task average):
- **Removing tactile input causes the largest degradation** — the benchmark genuinely requires contact.
- **Removing the Refinement Policy particularly hurts the perturbation setting**, since that is where online local correction is needed.
- Removing the **Subtask Planner** mainly costs long-horizon consistency.
- Removing the **Tactile World Model** weakens contact-aware goal conditioning.

Each component fails in the regime it was designed for — a cleanly separable hierarchy.

**Is the predicted tactile subgoal any good?** This is the most useful table, because it tests the world model directly rather than through downstream success. Ground truth is the actual tactile pressure observation when the robot reaches the annotated subgoal:

| Method | Temporal contact acc. | Contact IoU | Volumetric IoU |
|---|---|---|---|
| Copy current tactile (persistence) | 70.4 | 31.8 | 24.6 |
| Nearest-neighbour subgoal retrieval | 77.5 | 39.2 | 31.0 |
| **Tactile World Model** | **86.3** | **52.7** | **43.8** |

Two baselines that most tactile-prediction papers skip: persistence and retrieval. Beating persistence by 16 points of temporal accuracy and retrieval by 13.5 points of contact IoU is real evidence of prediction, not memorisation. But note the absolute numbers — contact IoU 52.7% means roughly half the predicted contact geometry is wrong.

**Planner analysis** — supervised subtask tuning beats zero-shot, and memory further improves phase-transition consistency. The headline: a **memory-augmented SFT Qwen3-VL-4B outperforms a zero-shot Qwen3-VL-32B**, so task-phase supervision and execution history matter more than model scale at this interface.

**Stated limitations.** Six tasks only; short-horizon subgoal prediction that struggles when occlusion or perturbation admits multiple plausible futures (uncertainty-aware or multi-candidate subgoals proposed as future work); tied to one sensing layout, with transfer to a different sensor or hand requiring calibration, normalisation and adaptation data; and a **fixed scheduling profile** — the planning rate, world-model refresh rule, chunk length and residual commit interval are hyperparameters they did not adapt.

## 7. What it adds that the others don't

The **multi-timescale decomposition**, and the argument behind it: appending tactile tokens to a VLA at the visual rate structurally mismatches contact dynamics, no matter how good the fusion is. It is also the only world model here evaluated against **persistence and retrieval** baselines on the prediction task itself, and one of very few to benchmark against another tactile foundation policy ([[ftp-1]]) rather than vision-only baselines. Contrast [[tacmamba]], which addresses the same fast-reflex/slow-reasoning split with a compression adapter instead of a hierarchy.
