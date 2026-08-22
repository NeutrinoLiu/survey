# TacWAM — Anchor-Guided World Action Model with Mechanics-Aware Tactile Prediction

**arXiv:2607.28391** · Tsinghua University + Manifold AI (Jin, Ma, X. Zhang, Gao, Wu, Y. Li) · Jul 2026

**One line.** Takes the opposite position to every other world-action model here: future tactile is a **training target only** — the action branch is *forbidden* from reading it — and the ablation showing what happens when you relax that is the most dramatic in the survey.

## 1. What "tactile" means here

Three heterogeneous gripper-side signals fused into one latent:

- **Tactile appearance** (the gel image)
- **Dense force fields** (through a Force CNN)
- **Deformation flow** (through a Flow CNN)

fused by the **Spatially Aligned Fusion (SAF) Encoder**: spatial alignment → modal interaction block → **bilateral fusion + pooling**, with a reconstruction head supervising **bilateral resultant force and torque** as a *global wrench* target.

The design logic is that local contact patterns and global wrench are different information and both are needed — dense fields preserve where and how, the wrench reconstruction preserves how much overall.

The authors are careful about what this latent is: *"we do not assume that `z^tac` is the full physical state. It is a tactile-observable representation of force- and deformation-related signals, used as the prediction space for future tactile modeling."* That is a more honest framing than most.

Their motivating examples are good: a stable-looking grasp may be close to slip; an apparently aligned insertion may already be mechanically constrained; a brittle object may look unchanged while force becomes unsafe.

## 2. Data curriculum

Four real contact-rich tasks covering three regimes: **fragile grasping** (chip grasping), **sustained surface contact** (whiteboard wiping), and **dynamic in-hand manipulation** (two-pen twirling). 20 real-world trials per condition.

## 3. Model

A visual WAM (Wan VAE video encoder, Video DiT) extended with a **Tactile DiT** and an **Action DiT**, plus a **tactile history encoder** `E_hist` producing a compact context `c^tac_{t0}` summarising recent interaction changes before the current action chunk. An auxiliary **contact-event loss** `L_contact` supervises the history pathway.

## 4. How tactile enters the model — Anchor-Guided Tri-Modal (AGT) Attention

The information structure is the contribution. Streams are organised around two **anchors**: the current visual frame `V_0` and the current observed tactile state `T_0`. Then:

- Future visual tokens and future tactile tokens are **prediction targets**
- **Action tokens may read only** `V_0`, `T_0`, and other action tokens

So future tactile supervises the shared representation through co-training, but **never reaches the action branch**. The stated principle: *"action generation remains restricted to information available at deployment."* At inference the mask collapses to `V_0` plus the tactile stream and action tokens.

This is a train/test-consistency argument rather than a modality-protection argument, and it is orthogonal to the [[n0-twam]] vs [[feelworld]] debate — TacWAM's concern is not that tactile will swamp vision but that **predicted tactile is a leak**: during training the action branch could exploit target-derived quantities that at deployment must be generated.

## 5. Experiment setup

Four tasks; vision-only VLA/WAM and visuo-tactile baselines. Three **nested** ablations that progressively remove history and relax attention restrictions, holding task data, action representation and the tactile-future objective fixed.

## 6. Does it work?

**Main result:** average success **75.0%**, exceeding the strongest evaluated baseline by **+37.5 points**, and better on every task.

**The staged ablation** (success over 20 real trials):

| Variant | Chip grasping | Whiteboard wiping | Average |
|---|---|---|---|
| **TacWAM** | **90.0%** | **75.0%** | **82.5%** |
| w/o tactile history | 50.0% | 60.0% | 55.0% |
| w/o history + **Attn-AT** (action reads full tactile future) | 30.0% | 45.0% | 37.5% |
| w/o history + **Attn-VT** (bidirectional visual↔tactile futures) | 10.0% | 5.0% | **7.5%** |

Read this top to bottom. Removing tactile **history** costs 27.5 points, concentrated on chip grasping (90 → 50), where the force build-up preceding a stable grasp cannot be inferred from the current anchor alone.

Then the two relaxations, both of which *add* information and both of which *hurt*:

- **Attn-AT** — letting action tokens read the full tactile prediction sequence `T_{0:H}` drops another 17.5 points. The diagnosis: during training the action tokens exploit **target-derived** future variables, whereas at deployment they receive **generated** counterparts. A train–test mismatch masquerading as extra context.
- **Attn-VT** — enabling bidirectional exchange between visual prediction tokens and tactile future tokens **nearly collapses the system to 7.5%** (5.0% on wiping). Fully bidirectional video–tactile future exchange destabilises multimodal co-training.

That last row is worth sitting with, because it is the strongest counter-evidence in this survey to [[n0-twam]]'s "let vision and touch stay fully attentive to one another" position. The two papers disagree, at different scales and with different backbones, and TacWAM's collapse is far more severe than any gain N₀-TWAM reports. The likely reconciliation is scale — N₀-TWAM has 30,000 hours and separate expert weights; TacWAM has four tasks and a shared attention — but nobody has tested it.

**Does tactile history improve the *prediction*, not just the policy?** They decode predicted tactile states to reconstruct the bilateral resultant wrench and compare force-magnitude curves against ground truth over complete episodes. TacWAM tracks both magnitude and temporal trend more closely than the history-free variant; the difference is clearest on whiteboard wiping, where the history-free prediction **lags** the ground-truth force changes. The interpretation is mechanistic and convincing: without recent tactile context the model must wait for additional incoming evidence before updating its contact-phase estimate. History removes that delay.

## 7. What it adds that the others don't

The **deployment-consistency constraint**, and the ablation proving it is load-bearing rather than conservative. Every other predictive tactile model here feeds predicted touch to the action branch ([[n0-twam]], [[dream-tac]], [[hitac-wam]] via ranking, [[tacforesight]] as an anticipatory prior); TacWAM shows that on small data this creates a shortcut that actively degrades deployment. The SAF encoder's pairing of dense local fields with global wrench reconstruction is also a useful template for turning heterogeneous tactile channels into one prediction space.
