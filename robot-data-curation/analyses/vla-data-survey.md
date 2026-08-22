# VLA Datasets, Benchmarks & Data Engines — Survey (framing reference)

## 0. Card

| Field | Value |
|---|---|
| **Work** | Vision-Language-Action in Robotics: A Survey of Datasets, Benchmarks, and Data Engines |
| **Authors** | Ziyao Wang, Bingying Wang, Hanrong Zhang, Tingting Du, Tianyang Chen, Guoheng Sun, Yexiao He, Zheyu Shen, Wanghao Ye, Ang Li |
| **Orgs** | University of Maryland College Park · University of Utah · others |
| **Date** | 2026-04-24 |
| **Artifact** | arXiv 2604.23001 (`paper.pdf`, `paper.html`); github.com/ziyaow1010/vla-datasets-benchmarks (continuously updated) |
| **Role here** | **Not a pipeline — the taxonomy this survey uses for framing.** |

## 1. The central claim

> *"Despite remarkable progress in Vision–Language–Action (VLA) models, a central bottleneck remains underexamined: **the data infrastructure that underlies embodied learning**. In this survey, we argue that **future advances in VLA will depend less on model architecture and more on the co-design of high-fidelity data engines and structured evaluation protocols**."*

And the closing prescription:
> *"Addressing them, we argue, requires **treating data infrastructure as a first-class research problem rather than a background concern**."*

This is the position that the present survey is testing empirically: across ~35 works from 2025–2026, how many actually treat curation as a first-class problem, and what do they do?

## 2. The three-pillar taxonomy

| Pillar | Definition | Sub-categories |
|---|---|---|
| **Datasets** | *"curated collections of demonstrations used for training VLA models"* | synthetic · real-world |
| **Benchmarks** | *"standardized evaluation protocols, task settings, and metrics"* | task setting (tabletop vs non-tabletop) · episode horizon · task complexity |
| **Data engines** | *"scalable systems or pipelines designed to collect, generate, or augment VLA training data"* | **video-to-data pipelines · hardware-assisted collection systems · generative data engines** |

The data-engine trichotomy maps cleanly onto the works in this survey:
- **Video-to-data**: EgoScale, ACE-Ego-0, HumanNet, PhysBrain, Open-AoE, DYNA-2, Qwen-RobotManip's H2R
- **Hardware-assisted collection**: EgoVerse (Aria), Open-AoE (smartphones), AGIBOT (G2 + force control), GR-3 (VR + compliance control), AXIS (browser teleop)
- **Generative**: RoboCurate, GR00T N1's neural trajectories, LingBot-VA 2.0's robot→human synthesis, Cosmos 3's synthetic mixture

## 3. The diagnoses that this survey's evidence bears on

**(a) Datasets — a fidelity–cost trade-off.**
> *"we categorize real-world and synthetic corpora along **embodiment diversity, modality composition, and action space formulation**, revealing a **persistent fidelity-cost trade-off that fundamentally constrains large-scale collection**."*

Confirmed repeatedly in the works surveyed here: Open-AoE trades sensing fidelity for smartphone accessibility; EgoVerse runs two streams (academic/controlled vs industry/scale) precisely because one point on the curve does not serve both purposes; DYNA-2 accepts a minimal two-channel action abstraction to make a million hours representable.

**(b) Benchmarks — structural gaps.**
> *"exposing **structural gaps in compositional generalization and long-horizon reasoning evaluation** that existing protocols fail to address."*

Independently reached by **Qwen-RobotManip**, which argues that *"most standard benchmarks systematically fail to capture the quality of pretraining"* and builds RoboTwin-IF and RoboTwin-XE in response. Also by **GR-3**'s Invalid-Tasks and Novel-Destinations settings, and **DYNA-2**'s insistence on external held-out evaluation sets.

**(c) Comparability is broken.**
> *"different works use **different task definitions, success criteria, and data splits**, which makes it difficult to compare methods or determine whether reported improvements reflect genuine generalization."*

**(d) Data engines share limitations.**
> *"identifying their **shared limitations in physical grounding and sim-to-real transfer**."*

Directly addressed by **RoboCurate** (simulator-replay action verification) and **LingBot-VA 2.0** (VLM physical-plausibility scoring) — two independent 2026 works concluding that generative data must be verified against physics.

## 4. The four open challenges — and where this survey finds answers

| Challenge (survey's framing) | Works in this collection that address it |
|---|---|
| **Representation alignment** | Qwen-RobotManip (canonical state-action + camera-frame delta), ABot-M0 (delta-EEF + axis-angle + pad-to-dual), ACE-Ego-0 (spatial/structural/temporal alignment), Hydra-0 (image-plane action flow) |
| **Multimodal supervision** | ACE-Ego-0 (reliability-aware graded loss), π₀.₇ (metadata conditioning), ABot-World-0 (hard-reject vs soft-weight), DYNA-2 (two-tier routing) |
| **Reasoning assessment** | RoboBrain, RxBrain, PhysBrain (structured QA supervision), GR-3 (invalid-task refusal) |
| **Scalable data generation** | GR00T N1 (88 h → 827 h), Qwen-RobotManip H2R (1,933 h → 24,808 h), RoboCurate (verified synthesis), AXIS (community engine), LingBot (robot→human) |

## 5. Why it is included
This survey supplies the **vocabulary and problem statement**; the other ~34 entries supply the **evidence**. Where the survey says the field *should* treat data infrastructure as first-class, the works collected here show which organizations actually do — and reveal that the most detailed disclosures come disproportionately from Chinese industry labs and open academic groups, while several frontier Western labs publish essentially nothing about their pipelines.
