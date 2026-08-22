# RATG — Representation-Aligned Tactile Grounding for Contact-Rich Robotic Manipulation

**arXiv:2607.14609** · Fudan University + Lenovo CTO + NTU + TeleAI China Telecom (R. Chen, J. Jia, T. Yang, X. Zhou, Q. Sun, J. Zhong, S. Zhang, N. Chen, B. He, W. Li, W. Zhang) · Jul 2026

**One line.** Asks a question nobody else asks — **where inside a VLA should the future-tactile loss attach?** — answers it with frozen linear probes rather than architectural intuition, and shows the answer is worth **16 points** over the obvious choice.

## 1. The question

Once you decide to supervise a policy with future tactile prediction, *"the key question is what representation it should shape."* Two intuitive answers, both argued to be wrong:

- **The final action representation** — tempting, since touch and motor control are tightly coupled. But *"the final action state is already specialized for immediate motor decoding and may have compressed away information useful for modeling future contact."*
- **The vision-language backbone** — *"may keep the objective too close to semantic perception and too far from the action-conditioned dynamics that give tactile signals their control relevance."*

Hence: *"tactile prediction is not simply an auxiliary loss to be added anywhere in the policy."*

## 2. The diagnosis — frozen linear probes

Rather than picking by convention, they **probe** where future tactile information is most linearly accessible along the action pathway. The finding, consistent **across settings with and without tactile input and across model scales**:

> **Intermediate action-expert representations are more predictive of future tactile states than either vision-language backbone features or terminal action states.**

Interpretation: *"future contact information is most available after action-conditioned interaction features have formed, but before the representation becomes specialized for immediate motor prediction."*

That is a clean, testable claim about the internal structure of a VLA, and it is the sort of measurement the tactile-VLA cluster has otherwise substituted with taste.

## 3. Model — the Latent Tactile Predictor

A lightweight head predicting **compact future tactile embeddings** from the diagnosed intermediate representation. Two properties:

- **Latent, not raw.** Raw tactile is *"high-dimensional, sensor-specific, and noisy, so a raw prediction loss may emphasize sensing artifacts rather than control-relevant contact dynamics."*
- **Training-time only** — *"without changing the inference pathway."*

## 4. Results

**Main** (five real contact-rich tasks: Plug Insertion, USB Drive Insertion, Wiping, Deformable, Bulb):

| Grounding interface | Avg success |
|---|---|
| VLM-side prediction | 58% |
| Final action-state prediction | 62% |
| **Intermediate action-expert (ours)** | **74%** |

**+16** over VLM-side, **+12** over final-action. And a detail that supports the mechanism rather than a simple proximity story: *"VLM-level prediction outperforms final-action-state prediction, while intermediate grounding remains best, suggesting that tactile prediction does not simply improve by moving closer to the motor output."* The relationship is non-monotonic in depth — which is exactly what "there is a specific right place" predicts.

**Backbone generalisation** (π₀ with LoRA, relative comparison within the π₀ family):

| Method | Avg success |
|---|---|
| π₀ | 40% |
| + tactile conditioning | 54% |
| + VLM-side prediction | 58% |
| + final-action prediction | 59% |
| **+ representation-aligned grounding** | **73%** |

The same ordering on a different backbone and adaptation protocol — *"the diagnosed intermediate action-expert interface is not an artifact of a single backbone."*

**Ablation 1 — latent vs raw target.** Replacing compact latents with direct raw tactile prediction drops Plug Insertion from **80% → 55%**. Raw prediction *"can force the grounding loss to account for sensor-level details, which may include measurement noise, calibration bias, and local contact artifacts."* Yet another independent confirmation of the theme running through [[tactile-wam]], [[tacgen]] and [[n0-vtla]]: **do not reconstruct raw tactile.**

**Ablation 2 — why not just supervise everywhere?** This is the decisive control:

| Variant | Avg success |
|---|---|
| None (tactile input, no future prediction) | 41% |
| **Multi-A** — all action-expert features | **38%** |
| Multi-B — every second layer (0/2/4/…) | 48% |
| Multi-C — middle layers 4–11 of 16 | 60% |
| **Ours** — single aligned interface | **74%** |

**Multi-A is worse than not predicting tactile at all** (38% vs 41%). Broadcasting the same contact-dynamics target across representations with different roles *"can introduce mismatched supervision."* And the monotone improvement A → B → C → ours as the supervision narrows toward the diagnosed layer is about as direct as evidence gets that the probe found something real.

## 5. What it adds that the others don't

A **diagnostic method for placing tactile supervision**, and the demonstration that placement is worth more than the presence of the supervision itself (41% → 74% by moving one loss to the right layer; 38% by putting it everywhere). Every other paper in the VLA cluster argues about *where tactile input enters* — the language prefix ([[n0-vtla]]), the action expert ([[forcevla2]]), a separate stream ([[modsensorystream]]) — RATG is the only one asking where the *supervision* should land, and the only one answering with probes instead of assertion.
