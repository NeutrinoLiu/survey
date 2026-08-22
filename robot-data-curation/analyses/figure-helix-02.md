# Figure Helix 02 + Project Go-Big — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Works** | **Project Go-Big** (2025-09-18) · **Helix 02** (2026-01-27 / 2026-02) |
| **Org** | Figure AI |
| **Artifacts** | `go-big.html`, `helix02.html` (figure.ai news posts) |
| **Disclosure level** | ⚠️ **C — company blog posts.** No corpus size, no pipeline description, no filtering criteria, no quantitative results tables. Strategy is clearly stated; methodology is not. |
| **Corpus** | Egocentric human video from **Brookfield's 100,000+ residential units**; System 0 trained on **1,000+ hours of human motion data** + large-scale simulation |
| **Stance** | **Acquire data by acquiring access to environments.** The scaling lever is real-estate partnership, not algorithms. |

## 1. Thesis — the missing pretraining corpus

> *"The biggest breakthroughs in machine learning have come from pretraining large neural networks on massive, diverse datasets: ImageNet for vision, Wikipedia for language, or YouTube for generative video models. **Unlike vision or language, robotics lacks a large-scale equivalent—no 'YouTube for robot behaviors.'**"*

And the structural argument for why humanoids specifically can borrow human video:
> *"Humanoid robots… offer a unique structural advantage: **their perspectives and kinematics mirror our own**, making it possible to transfer knowledge directly from everyday human video."*

This is the cleanest statement of the morphology argument that underpins the entire human-video wing of this survey (EgoScale, DYNA-2, HumanNet, EgoVerse, Ψ₀). Figure's version is the strongest form: build the robot to match the data, rather than transforming the data to match the robot.

## 2. The data-acquisition strategy — Project Go-Big

The distinctive move is not a pipeline but a **partnership**:

> *"Figure is building the world's largest and most diverse humanoid pretraining dataset, accelerated by an unprecedented partnership with **Brookfield**… Brookfield's $1 trillion global asset base—**over 100,000 diverse residential units, 500 million square feet of commercial office space, and 160 million square feet of logistics space**—will help accelerate Project Go-Big by capturing human goal-directed behavior at an unprecedented scale and diversity of real-world environments."*

Two properties worth naming:
- **Collection is passive.** Data is *"collected passively as people do behaviors in real Brookfield homes"* — no teleoperation, no staged tasks, no operator cost per hour.
- **Environment diversity is purchased, not sampled.** Where EgoVerse assembles 240 scenes through a consortium and HumanNet crawls the web, Figure obtains legal access to 100,000 distinct real homes. Scene diversity — which EgoVerse's own ablation identifies as the axis that matters most past a threshold — becomes a procurement outcome.

The implicit curation argument: *"Traditionally, teaching robots new skills required costly demonstrations, hand-coded programs, or tightly staged environments that **fail to capture the messiness of the real world**."* The messiness is the point, so there is little to filter.

## 3. The result claimed — 100% human video, zero robot demonstrations

> *"Using **100% egocentric human video data**… we trained Helix to translate human navigation strategies directly into robot control. Remarkably, **this approach required no robot demonstrations whatsoever**."*

- **Speech-to-nav**: responds to conversational commands ("Walk to the kitchen table", "Go water the plants"), *"autonomously generating closed-loop control from pixels to navigate complex, cluttered home environments."*
- **A single unified model**: *"One Helix network now outputs both high rate dexterous manipulation and navigation commands."*

Figure calls this *"a first in humanoid robotics."* It is the same class of claim as DYNA-2's zero-robot-data pretraining, arrived at independently and about a year earlier, though on navigation rather than manipulation and without a scaling curve.

## 4. Helix 02 and System 0

Helix 02 is a **unified whole-body loco-manipulation VLA system**. The relevant data fact:
- **System 0 (motion prior)** trained on **1,000+ hours of human motion data** plus large-scale simulation.

Prior Helix work covered upper-body manipulation (laundry folding, dishwasher loading, package reorientation); Go-Big extends the data story to navigation and whole-body behavior.

## 5. What is not disclosed
- **No corpus size** for Go-Big — no hours, episodes, or environments actually captured to date.
- **No processing pipeline**: no hand-pose extraction method, no filtering, no quality gating, no annotation scheme.
- **No action-label derivation** — how passive human video becomes navigation supervision is not described.
- **No quantitative results**: success rates, baselines, and ablations are absent.
- **No scaling evidence.** The claim is capability, not predictability.
- Privacy/consent handling for passively collected residential video is not addressed in the posts.

## 6. Position in the survey
Figure occupies a distinctive corner: **maximum data-access ambition, minimum published methodology.** It is included because the *strategy* is consequential and widely imitated — several 2026 works (DYNA-2, GEN-0/1, HumanNet) rest on the same premise that passively-captured human video is the scaling substrate — but nothing here is reproducible, and the entry should be read as a statement of industrial direction rather than as a documented pipeline.

## 7. Transferable takeaways
1. **Environment access is a data strategy.** Scene diversity, the axis EgoVerse shows matters most, can be obtained through partnership rather than through sampling or synthesis.
2. **Passive collection removes the per-hour operator cost** that bounds teleoperation corpora — the same argument DYNA-2 makes from first principles.
3. **Co-design morphology with the data source.** If the robot's kinematics and viewpoint match the human's, most of the retargeting pipeline other works build becomes unnecessary.
4. **Treat "messiness" as signal.** Staged environments are a curation choice with a generalization cost.
5. ⚠️ **Counter-lesson:** the absence of any published pipeline, corpus size, or evaluation makes this the least verifiable entry in the survey. Strategy claims of this kind should not be read as evidence about what works.
