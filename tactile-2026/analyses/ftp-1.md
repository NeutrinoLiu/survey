# FTP-1 — A Generalist Foundation Tactile Policy Across Tactile Sensors for Contact-Rich Manipulation

**arXiv:2606.13102** (v2) · Tsinghua + Shanghai Qi Zhi + **Sharpa** + SJTU + UC Berkeley + ETH Zurich + Fudan + Shanghai Innovation Institute (C. Yuan, Z. Zhang, M. Zhou, … K. Zhang, Y. Gao) · Jun 2026 · [site](https://ftp1-policy.github.io/)

**One line.** The tactile analogue of a vision-language foundation policy: **~3,000 hours across 21 sensors** aggregated into one **morphology-aware token space**, with the transfer claim validated by an ablation that separates "better data distribution" from "transferable tactile knowledge."

## 1. What "tactile" means here — a functional-area token space

**MTTS (Morphology-Aware Tactile Token Space)** organises any tactile signal into **24 functional-area slots**, one token each:

| Slots | Assignment |
|---|---|
| 0–14 | in-hand tactile — hand functional regions |
| 15–20 | force/torque from wrists and fingers |
| 21–23 | reserved |
| *(parallel grippers)* | two gripper-side sensors → **thumb-tip (slot 0)** and **index-fingertip (slot 1)**, reflecting their two-finger grasping *function* |

A learnable **functional-area embedding shared across all sensors** is added before the tokens enter the network, with **separate embeddings for left/right hands**.

The key idea is that the unifying abstraction is **anatomical function, not signal format**. A parallel gripper's two pads are represented as a thumb and an index fingertip because that is what they *do*. This is a different answer to heterogeneity from physical units ([[uniforce]], [[n0-twam]]), latent contrastive alignment ([[htt]], [[tactx]]), or explicit sensor conditioning ([[unitac]]).

**Heterogeneous encoders** then map each functional area into MTTS by observation type:

| Type | Example | Encoder |
|---|---|---|
| **Image** | GelSight | lightweight sensor-specific ViT → **shared pretrained T3 Transformer** → `[CLS]` token |
| **Array** | Contactile | CNN capturing spatial structure, compressing each area to one token |
| **State** | force/torque | **Fourier encoding** of raw state → lightweight MLP |

For sensors with multiple functional areas of the same shape, the encoder is **shared across those areas**, reducing sensor-specific parameters and encouraging common tactile dynamics to be modelled consistently.

## 2. Data curriculum — FTP-1-Dataset

| | |
|---|---|
| Duration | **~3,000 hours** |
| Sources | **26** |
| Sensors | **21** |
| Composition | 50% dexterous hand, 30% gripper, 20% human |
| Modalities | image, array, state — all standardised through MTTS |

Contributed data includes Sharpa North, EgoTac egocentric human data with AetherGlove, and PaXini's OmniSharingDB.

## 3. Model

Built on **π₀.₅**, multi-expert:
- **VL expert** — SigLIP vision + language tokenizer, pretrained vision-language Transformer
- **Tactile expert** — a **shared tactile Transformer** jointly modelling MTTS tokens across all sensors
- **Action expert** — flow matching, attending to VL expert outputs
- Proprioception fused via **adaptive RMSNorm**
- Output in a **Unified Action Space** handling control-signal heterogeneity

The explicit contrast with prior tactile VLAs: rather than *"inject[ing] tactile inputs into the vision-language expert via lightweight adapters"*, FTP-1 gives touch its own expert — the same architectural position as [[n0-twam]] and the opposite of [[at-vla]]/[[tacmodfusion]].

## 4. Experiment setup — distributed evaluation

Unusual and worth noting: pretrained checkpoints were **distributed to 5 independent institutions across global regions**, each finetuning on its own hardware, covering 4 tactile sensors and **14 tasks** — in-hand adjustment, force-controlled pressing, insertion and extraction, long-horizon dexterous manipulation.

Seen setups: UniVTAC, Sharpa North, Sharpa & Dexmate. **Unseen** setups: **FlexivXense**, **TactileUMI**.

## 5. Results

**Seen sensors: +17.2%** average success.

**Unseen sensors** (success rate):

| Method | FlexivXense (Insert Hanoi, Insert USB) | TactileUMI (Wipe Board) | **Avg** |
|---|---|---|---|
| π₀.₅ | | | 15.0 |
| Tactile-VLA | | | 8.3 |
| FTP-π₀.₅ | | | 15.0 |
| **FTP-1** | | | **46.6** |

**+31.6 points** on sensors never seen in pretraining. Note that **Tactile-VLA (8.3) does worse than vision-only π₀.₅ (15.0)** — and that π₀.₅ and FTP-π₀.₅ tie, which the authors read correctly: *"simply adding a tactile branch does not necessarily help without an appropriate modality-fusion prior and pretraining knowledge."*

**Behavioural evidence** is more convincing than the aggregate. On **Insert Hanoi**, FTP-1 and π₀.₅ show recovery behaviours the other baselines lack (attributed to large-scale pretraining), and FTP-1 additionally shows **reactive insertion control**: *"when the circular Hanoi piece is misaligned with the pillar, FTP-1 slows down the insertion motion based on tactile feedback, whereas π₀.₅ does not."* On **Insert USB** with only 100 demonstrations, baselines *"exhibit small shaking motions during insertion"*. On **Wipe Board**, other models *"struggle to maintain stable pressing force and may lose tight contact."*

## 6. The ablation that matters — is it the data or the knowledge?

The authors pose the two hypotheses explicitly:

- **H1 (Data Distribution)** — gains come from FTP-1's pretraining data being closer to the downstream distribution.
- **H2 (Transferable Knowledge)** — the tactile branch learns genuinely transferable manipulation knowledge.

The control: pretrain **NTP** with *identical data and optimisation but no tactile inputs and no tactile architecture*, then add the same tactile architecture at finetuning (**NTP-1**). Everything is identical except which checkpoint the tactile branch starts from.

- On **UniVTAC** (seen), NTP-1 beats FTP-π₀.₅ — so **H1 is real**, the dataset distribution does help. But NTP-1 still **underperforms FTP-1**.
- On **FlexivXense** (unseen), FTP-1 beats NTP-1 by **+37.5%**. Without tactile-branch pretraining, NTP-1 *"shows much less robustness to tactile changes and produces unstable actions during key insertion stages."*

This is the right experiment, and it is one of the few places in this survey where a foundation-model paper actually isolates whether its pretraining *of the new modality* is doing the work. The answer: partly H1, decisively H2 on unseen sensors.

## 7. Stated limitations

- FTP-1 *"mainly focuses on general tactile perception and does not yet address tactile- or force-based servoing and control"* — no predictive or low-level control, unlike [[n0-twam]] or [[tacforesight]].
- *"the scale and diversity of our pretraining dataset remain limited"* — 3,000 hours is small against vision-language pretraining, and against [[n0-twam]]'s 30,000.

## 8. What it adds that the others don't

**Function as the unifying abstraction**, and the first genuine cross-sensor *policy* foundation model rather than a representation. Mapping a two-finger gripper onto thumb and index slots is a small idea with large consequences: it lets human hand data, dexterous hand data and gripper data occupy the same token space without format conversion. The **NTP-1 control** and the **five-institution distributed evaluation** are also the strongest methodological practices in the foundation-model cluster — the former proves the claim, the latter makes the result hard to over-fit to one lab's hardware.
