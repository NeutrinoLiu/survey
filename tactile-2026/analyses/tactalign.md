# TactAlign — Human-to-Robot Policy Transfer via Tactile Alignment

**arXiv:2602.13579** · University of Michigan + Nvidia + Amazon FAR + UC Berkeley + UW + **Microsoft Research** (Wi, Yin, Xiang, Sharma, Malik, Mukadam, Fazeli, Hellebrekers) · Feb 2026 · [site](https://yswi.github.io/tactalign/)

**One line.** Transfers human glove tactile to robot tactile **without paired data**, using a rectified flow trained on noisy *pseudo-pairs* derived from hand-object interaction — because strict human-robot correspondence is exactly what contact-rich manipulation makes impossible to maintain.

## 1. The problem with paired data

Existing human-to-robot tactile transfer methods *"assume identical tactile sensors, require paired data, and involve little to no embodiment gap."* The concurrent UniTacHand does address cross-sensor transfer but *"relies on strict spatiotemporal correspondence between human and robot throughout task demonstrations."*

The objection is physical, not practical: *"This strict pairing can be prohibitively difficult to maintain during contact-rich interactions involving sliding contact or dynamic object motion necessary for general manipulation."* The moment the object slides, the human's and robot's contacts diverge.

Cross-sensor transfer methods more broadly *"primarily focus on static contact scenarios and coarse categorical alignment objectives"* — leaving continuous tactile reasoning during dynamic interaction unaddressed. That is a fair critique of [[tactx]] (quasi-static grasps) and [[uniforce]] (quasi-static equilibrium), both of which acknowledge the same limitation.

## 2. Method — rectified flow over pseudo-pairs

Two stages:

1. **Independent self-supervised pretraining** of human and robot tactile encoders → modality-specific latents.
2. **Cross-sensor alignment by rectified flow**, transporting glove latents into the robot tactile latent space, trained on **pseudo-pairs derived from hand-object interactions** rather than explicit pairing.

Why rectified flow specifically: it *"is well suited to learning mappings under noisy pseudo-pairs arising from the non-unique relationship between hand-object motion and tactile observations."* The same hand-object configuration can produce many tactile readings; a deterministic regressor would average them, a flow transports the distribution.

**Pseudo-pair construction** uses a similarity metric `S(O_i^h, O_j^r)` with a threshold δ excluding degenerate matches. The sensitivity analysis is reassuring:

| δ | 1.5 | 2.0 | 2.5 | 3.0 | 3.5 |
|---|---|---|---|---|---|
| EMD reduction (%) ↑ | 82.7 | 83.2 | 83.9 | 80.7 | 83.9 |

Essentially flat over a 2.3× range — the method is not threshold-tuned. (Contrast [[tactx]], whose binary-contact baseline was *"sensitive to the contact threshold."*)

Contact thresholds are chosen from the raw-signal norm histograms, which show a *"high-density peak near zero"* for non-contact and a clean separation on contact: **δ_h = 1200** for the human glove, **δ_r = 30** for the robot Xela sensor. That 40× difference in raw units is a compact illustration of why cross-sensor transfer is hard.

**Cost:** the alignment module is an MLP with three 1024-wide hidden layers, 100 discretised time steps, 200,000 epochs, lr 5e-5 — **about 10 minutes on a single RTX 4090.**

## 3. Experiment setup

Four contact-rich tasks with per-task finger selection, which is a nice detail:

| Task | Fingers used | Action space |
|---|---|---|
| **Pivoting** | index only | 6-D: index fingertip position + wrist rotation |
| **Insertion** | thumb, index, middle | 6-D, wrist motion only |
| **Lid closing** | thumb, index, middle, ring | 6-D, wrist motion only |
| **Light-bulb screwing** | all four fingertips as input *and* output | no wrist rotation — wrist stationary |

Inference at **10 Hz** with action chunk 32 (executing 4 / 2 / 8 actions per rollout for pivoting / insertion / lid closing), and **30 Hz executing 12 actions** for screwing, which needs a higher control rate.

Human data budget: **≤ 5 minutes** per task.

## 4. Results

| Comparison | Gain |
|---|---|
| H2R co-training vs. **no tactile** | **+59%** |
| vs. **no alignment** (tactile present, unaligned) | **+51%** |
| Human-only objects vs. robot-only | +59% |
| **Unseen** objects vs. robot-only | +54% |
| Light-bulb screwing, **zero-shot** H2R | **+100%** over no-tactile / no-alignment |

The **+51% over no alignment** is the load-bearing number: it separates "human tactile data helps" from "aligned human tactile data helps," and shows unaligned cross-sensor tactile contributes almost nothing.

Two further points worth noting. **The same learned alignment is reused for policies on unseen tasks** — it is a sensor-pair property, not a task-specific mapping, so the 10-minute training cost amortises across everything. And light-bulb screwing is **zero-shot**: no robot demonstrations at all, only aligned human tactile.

## 5. What it adds that the others don't

**Alignment without pairing.** Every other cross-sensor method in this survey requires synchronised observations of the same contact — [[htt]] by UMI collection, [[tactx]] by different-sensor-per-finger, [[uniforce]] by force equilibrium, [[taf-vla]] by an indenter rig. All are quasi-static by necessity. TactAlign relaxes the requirement to *unpaired demonstrations of the same task*, which is the only formulation that survives sliding contact and dynamic object motion — and does it across a genuine embodiment gap (human hand → robot hand) rather than sensor-to-sensor on the same gripper. The rectified-flow choice, justified by the one-to-many nature of hand-motion-to-tactile, is the right tool for the noise structure of pseudo-pairs.
