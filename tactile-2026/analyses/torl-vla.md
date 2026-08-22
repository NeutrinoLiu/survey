# TORL-VLA — Tactile Guided Online Reinforcement Learning for Contact-Rich Manipulation

**arXiv:2606.09337** (v3) · Meituan + BIT + Beihang + CASIA + CUMTB (H. Zheng, Y. Yang, K. Ma, S. Xu, T. Xie, G. Li, X. Wang, Y. Ma, S. Liu, Mao, B. Liu) · Jun 2026 · [site](https://torl-vla.github.io/)

**One line.** Draws the distinction the tactile-VLA field mostly elides — **"contact awareness is not equivalent to contact adaptation"** — and adds online RL so the policy can correct at contact bottlenecks, with a credit-assignment fix for learning from human takeovers.

## 1. What "tactile" means here

**Tactile-derived wrench** — fingertip tactile converted to a wrench, used in three forms:
- **wrench history token (WHT)** — temporal encoding of measured wrench
- **future wrench prediction (FWP)** — an auxiliary objective producing predicted wrench cues
- **measured wrench feedback** at execution time

## 2. The argument

Existing tactile-VLAs add tactile as inputs, fusion features, or auxiliary targets in supervised training, improving contact *awareness*. But *"most tactile-aware VLAs remain static offline policies: once execution enters out-of-distribution contact states, they lack a mechanism to correct actions using current tactile feedback."*

Then the safety problem with the obvious fix: relying entirely on policy exploration for online RL is costly and unsafe in contact-rich tasks. **Human intervention** is needed when the robot is about to fail or produce unsafe contact — but this creates a **value-learning bias**: if a poor policy action leads to human takeover and the rollout is later completed after intervention, a standard value function **wrongly credits the preceding policy action for the final success**.

## 3. Model — three stages

**Stage I — Tactile-Aware VLA training.** Tactile-derived wrench sequences are fused **after** visual-language encoding via **MoE routing** (T&VL-MoE, router network over K experts), predicting both action references and future wrench sequences. Includes a **zero-initialised physical bypass** — a direct wrench-token path around the fusion.

**Stage II — online reference adaptation.** The **frozen** wrench-aware VLA supplies vision tokens, reference actions and predicted wrench. A lightweight **stage-specific actor-critic** refines actions online, conditioned on VLA reference actions, measured wrench, predicted wrench cues and compact visual-semantic tokens. A **stage estimator** routes to the right actor-critic.

**Two critics:**
- **Task critic** — standard
- **Intervention-censored critic** — blocks post-intervention success from propagating back across human-correction boundaries

**Stage III** — real-world deployment.

## 4. Experiment setup

Long-horizon real-robot contact-rich tasks: **cup placement into a narrow holder**, **latch opening/closing**, **fragile egg handling**. Metrics include subtask success, full-task success, and **time-bounded execution efficiency** (throughput per 10 minutes) — the latter rare and useful, since a policy that succeeds by retrying indefinitely is not a good policy.

## 5. Results

| Method | Sub 1 ↑ | Sub 2 ↑ | Sub 3 ↑ | **Full-task ↑** | Avg time (s) ↓ |
|---|---|---|---|---|---|
| π₀.₅ | 18/30 | 15/30 | 20/30 | **12/30** | 199.65 |
| TA-VLA | 19/30 | 17/30 | 20/30 | 12/30 | 204.45 |
| ForceVLA | 21/30 | 20/30 | 22/30 | 15/30 | 195.34 |
| TORL-VLA w/o RL | 25/30 | 23/30 | 25/30 | 21/30 | 191.91 |
| RLT | 26/30 | 25/30 | 25/30 | 23/30 | 175.23 |
| **TORL-VLA** | **30/30** | **29/30** | **30/30** | **28/30** | **165.45** |

Full-task success goes **12/30 → 28/30**. Note how the gap widens from subtask to full task: on the subtasks the offline baselines are at 60–73%, but compounding across a long horizon leaves them at 40–50% end to end. Online adaptation is what closes that.

**A revealing failure-mode difference** on the egg task: **3 of RLT's 5 failures are overly aggressive grasps that break the egg**, whereas TORL-VLA w/o RL fails mainly on *imprecise grasping positions*. Different methods fail in qualitatively different ways, and the RL variant's failures are the dangerous kind — an argument for [[vitar]]'s bounded-residual position.

**Reference-model ablation:**

| Configuration | Cup ↑ | Latch ↑ | Egg ↑ |
|---|---|---|---|
| w/o WHT (wrench history) | 24/30 | 22/30 | 22/30 |
| w/o FWP (future wrench pred.) | 25/30 | 21/30 | 24/30 |
| **w/o MoE** | **18/30** | **17/30** | **19/30** |
| w/o Bypass | 23/30 | 20/30 | 21/30 |
| Full w/o RL | 25/30 | 23/30 | 25/30 |

**Removing MoE causes the largest degradation** — routing wrench features with visual-language representations matters more than either temporal encoding or future prediction. Removing the **zero-initialised bypass** also hurts, especially on Latch and Egg, suggesting the direct wrench-token path preserves fine-grained contact information the fusion pathway loses. WHT and FWP are complements, not load-bearing.

**Online adaptation ablation:**

| Configuration | Cup ↑ | Latch ↑ | Egg ↑ |
|---|---|---|---|
| w/o wrench context | 27/30 | 27/30 | 26/30 |
| w/o IC critic | 27/30 | 26/30 | 28/30 |
| **Full** | **30/30** | **29/30** | **30/30** |

The intervention-censored critic's largest drop is on **Latch — the hardest subtask, where interventions are most frequent** and credit assignment across correction boundaries matters most. The adaptation curves show it also improves **early data efficiency**, giving faster gains in both success rate and throughput per minute of online data.

**Stated limitations.** Evaluation is limited to latch-box manipulation with a few bottlenecks; TORL-VLA depends on **reliable fingertip wrench measurement**, vulnerable to sensor noise, calibration error, coordinate-frame misalignment and hardware drift; and intervention-based learning **assumes human interventions reliably indicate imminent failure**.

## 6. What it adds that the others don't

The **intervention-censored critic** — a specific, generalisable fix for a credit-assignment bug that arises whenever a robot learns online with a human safety net, which is the only realistic way to do online RL on contact-rich tasks. Plus the **awareness vs. adaptation** framing, which cleanly separates this work from the supervised-fusion majority ([[at-vla]], [[tacvla]], [[taf-vla]]) and from the offline-improvement approaches ([[taco]], [[n0-vtla]]'s ALTER). And the **throughput metric** — successes per 10 minutes — is a better measure of a contact-rich policy than success rate alone.
