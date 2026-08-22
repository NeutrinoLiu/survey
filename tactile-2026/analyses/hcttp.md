# TTP — Human-Centric Transferable Tactile Pre-Training for Dexterous Robotic Manipulation

**arXiv:2607.01067** · Peking University + **BeingBeyond** + Tsinghua (C. Zhang, Cai, Xi, H. Yuan, H. Luo, W. Zhang, S. Zheng, C. Xu, Z. Lu) · Jul 2026 · [site](https://beingbeyond.github.io/TTP/)

**One line.** Pretrains a VLA on **160 hours of egocentric *human* video with real tactile**, then post-trains on robots — with the pre-train and post-train setups held **strictly identical** to avoid the distribution mismatch that undermines most tactile post-training.

## 1. What "tactile" means here

Human hand tactile from instrumented gloves, mapped onto **MANO** and standardised into a **unified tactile space** that spans embodiments (human hand, Inspire hand, DexBotic hand, parallel gripper). Paired with a **unified action space**.

The consistency argument is the design's spine: *"The model is subsequently post-trained on downstream robotic tactile tasks while maintaining strict consistency with the pre-training setup, avoiding the pre-train/post-train distribution mismatch in previous literature."*

## 2. Data curriculum — H-Tac

| | |
|---|---|
| Duration | **160 hours** egocentric human video |
| Tasks | **300+** |
| Episodes | **135,000** |

Scale context from the paper's own comparison of tactile-annotated trajectory counts: VTDexManip ~10,000 · OmniViTac 21,879 · **H-Tac 135,000**. Roughly 6× the largest prior tactile-action corpus, obtained by instrumenting humans rather than robots.

The motivation is the standard cost asymmetry, stated sharply: *"Tactile sensors on different embodiments (especially dexterous hands) are non-unified in hardware integration, and teleoperating robots for contact-rich tasks are labor-intensive and hard to scale."*

## 3. Model — a dual-expert VLA

Built on a shared-attention multi-expert backbone: **understanding expert**, **tactile expert**, **action expert**, each with its own FFN and QKV, joined by shared attention over `<tac.> <state> <lan.> <vis.>` tokens.

- **Action expert** → future action chunks
- **Tactile expert** → **future tactile prediction**, explicitly modelling contact dynamics
- **Tactile-Action MPG** (motion prediction guidance) as an auxiliary objective

The stated aim: *"encourage the model to balance semantic reasoning and physical interaction."*

## 4. Results

**Real-robot, in-distribution** (10 trials; Peeling in cm of skin peeled, PaperFolding as % folded, others success %):

| Task | π₀.₅ | π₀.₅+tactile | BeingH-0.5 | TTP w/o pretrain | **TTP** |
|---|---|---|---|---|---|
| Peeling (Inspire) | 10.63 | 9.27 | 12.49 | 14.65 | **23.33 cm** |
| VaseWiping (single) | 30% | 50% | 50% | 70% | **100%** |
| VaseWiping (bimanual) | 50% | 40% | 70% | 60% | **90%** |
| Peeling (Gripper) | 10.39 | 12.02 | 11.37 | 14.48 | **15.24 cm** |
| PickPlaceChips | 10% | 20% | 10% | 60% | **80%** |
| PaperFolding | 0% | 4% | 12% | 57% | **84%** |
| SoftHard | 50% | 60% | 40% | **80%** | **80%** |
| PlugIn (Gripper) | 0% | 0% | 0% | 0% | **20%** |
| PlugIn (DexBotic) | 0% | 0% | 0% | 10% | 10% |

**Out-of-distribution** (5 trials) is where the pretraining shows:

| Task | π₀.₅ | π₀.₅+tactile | BeingH-0.5 | TTP w/o pretrain | **TTP** |
|---|---|---|---|---|---|
| Peeling (Inspire) | 5.74 | 5.48 | 9.28 | 11.25 | **19.12 cm** |
| VaseWiping (single) | 20% | 20% | 40% | 80% | **100%** |
| PickPlaceChips | 0% | 0% | 0% | 40% | **60%** |
| PaperFolding | 0% | 0% | 11% | 24% | **87%** |
| PlugIn (Gripper) | 0% | 0% | 0% | 0% | **20%** |

Two observations. **PaperFolding OOD**: TTP w/o pretrain falls from 57% → 24% under shift while full TTP goes 84% → **87%** — the human pretraining is what carries generalisation, not the architecture. And **π₀.₅ + tactile is repeatedly *worse* than plain π₀.₅** (peeling 10.63 → 9.27, bimanual wiping 50% → 40%, OOD peeling 5.74 → 5.48), yet another instance of naive tactile injection hurting, consistent with [[at-vla]], [[tactile-wam]] and [[ftp-1]].

The **PlugIn** rows are a useful reality check: everything except TTP scores exactly 0%, and TTP reaches only 10–20%. Precise insertion with a dexterous hand remains essentially unsolved.

**Pre-training ablations** (motion prediction error on validation, lower better):

| Variant | MPJPE | PA-MPJPE | MPJAE | PA-MPJAE |
|---|---|---|---|---|
| w/o MPG, w/o tac-pred | 25.5850 | 0.8622 | 0.0277 | 0.0620 |
| w/o MPG | 24.7597 | 0.8151 | 0.0267 | 0.0598 |
| w/o tac-pred | 24.5518 | 0.8009 | 0.0263 | 0.0583 |
| **TTP** | **23.5711** | **0.7877** | **0.0257** | **0.0559** |

**Data scaling** (10% → 100% of pretraining data): MPJPE **33.19 → 23.57**, PA-MPJPE **1.4066 → 0.7877**, monotone throughout — tactile pretraining scales, and has not saturated.

## 5. What it adds that the others don't

The **human-side scaling route with strict pre/post-train consistency**. [[egotac]] extracts tactile labels from human video by prediction; TTP *measures* them at 135k-episode scale and shows the resulting priors transfer to four different robot embodiments. The unified tactile *and* action spaces maintained across both phases is the specific discipline that makes the transfer work, and the OOD PaperFolding contrast (24% vs 87%) is the cleanest isolation of what human pretraining buys. Read alongside [[ftp-1]], which aggregates 3,000 hours across 21 sensors including 20% human data — TTP is the pure-human, single-glove version of the same bet.
