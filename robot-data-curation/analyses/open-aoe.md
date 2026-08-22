# Open-AoE — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Open-AoE: An Open Egocentric Manipulation Dataset and Toolchain for Embodied Learning |
| **Org** | Ant Digital Technologies, Ant Group |
| **Date** | 2026-07 |
| **Artifact** | arXiv 2607.14183 (`paper.pdf`, `paper.html`); github.com/ant-research/Open-AoE |
| **Disclosure level** | **A — full paper; the cloud processing pipeline itself is released** |
| **Corpus** | **~2,000 hours**, **500+ contributors**, **400+ smartphone models** |
| **Stance** | **Commodity smartphones + released pipeline.** Accessibility is the design constraint. |

## 1. Thesis — the gap is infrastructure, not video

> *"The community does not simply need more egocentric video. It needs **open data infrastructure** that can continuously collect, systematically process, and directly support model training."*

The dilemma it identifies:
- **Specialized hardware** (head-mounted rigs, AR/VR, Project Aria) *"can improve pose and camera sensing, but it narrows accessibility relative to commodity-smartphone collection."*
- **Large-scale passive video** is easy to get but *"usually lacks the hand motion, camera trajectory, action boundaries, and physical structure required for robot training."*
- Existing per-dataset pipelines *"remain fragmented and have yet to form a unified infrastructure."*

Open-AoE's bet: **take the accessibility, and pay for the missing signals with a heavier processing pipeline.**

## 2. Capture

- **~2,000 hours**, natural environments
- **500+ contributors**, **400+ smartphone models** — deliberately not a controlled device fleet
- Built on the **AoE consumer-smartphone collection framework**

The paper's Table 1 compares egocentric datasets on capture device, and notes honestly that many aggregated corpora *"without a verifiable model inventory are marked Mixed / N/A"* — device provenance treated as a first-class dataset property.

## 3. Cloud-side processing pipeline (released)

Raw smartphone clips → structured samples, through:

| Stage | Notes |
|---|---|
| **Quality screening** | First gate on uploaded clips |
| **Privacy erasure** | Run *inside* the pipeline — necessary when 500+ strangers upload home video from personal phones. One of only two works here (with HumanNet) treating privacy as a pipeline stage |
| **Temporal segmentation** | Split into action units |
| **Semantic labeling** | Text descriptions |
| **Camera-trajectory estimation** | Recovers the egomotion smartphones don't log |
| **Hand reconstruction** | **MANO-based hand poses** |
| **Atomic-action annotation** | Temporally localized atomic actions |
| **Multi-stage quality inspection** | Repeated QC rather than a single pass |

Released signals per sample: **video · text descriptions · MANO hand poses · camera trajectories · atomic action segments.**

The heterogeneity problem here is distinctive: **400+ different phone models** means 400+ intrinsics, rolling-shutter behaviors, stabilization pipelines, and frame-rate policies. Camera-trajectory estimation is doing the work that a calibrated rig would otherwise provide.

## 4. Downstream consumption toolchain (separately released)

Deliberately decoupled from production, so consumers can re-derive rather than accept:
- **Synchronized visual inspection** (browser-based)
- **4D hand–object interaction reconstruction**
- **Cross-embodiment motion retargeting**
- **Robotized-video generation** (rendering robots into human video, à la Qwen-RobotManip's H2R)

## 5. Training-ready toolchain

Maps aligned **video + hand + camera + language** signals into action representations and training interfaces for **three** model families:
1. **VLA policies**
2. **World-Action Models (WAMs)**
3. **World Models**

Supporting three consumption modes from one corpus is unusual and directly reflects the field's split (see the survey overview): the same egocentric hour can be an action label, a video-prediction target, or a world-model rollout, and Open-AoE refuses to pick one.

## 6. What they do not do
- Quality-screening and inspection criteria are not published as numeric thresholds.
- No dedup across contributors.
- No published scaling curve or downstream policy benchmark of its own — it is infrastructure, validated by usability rather than by a trained model.
- Smartphone capture yields weaker pose/camera sensing than Aria/Vision Pro; the accuracy cost versus EgoDex-class data is not quantified.

## 7. Transferable takeaways
1. **Release the production pipeline, not just the data.** A corpus is a snapshot; a pipeline is a capability.
2. **Separate production from consumption toolchains**, so downstream users can re-derive representations instead of inheriting your choices.
3. **Privacy erasure belongs in the pipeline** when collection is crowd-sourced from personal devices.
4. **Record device provenance** — with 400+ phone models, the device inventory *is* metadata.
5. **Serve multiple consumption modes** (VLA / WAM / World Model) from one annotated corpus rather than committing to one action representation.
6. Read against **EgoVerse** (consortium + EgoDB, controlled protocol, Aria-class sensing) — Open-AoE and EgoVerse are the two "living dataset" designs in this survey, taking opposite positions on the hardware-quality vs. accessibility tradeoff.
