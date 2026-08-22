# Cosmos-Predict2.5 / Cosmos-Transfer2.5 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | World Simulation with Video Foundation Models for Physical AI (Cosmos-Predict2.5, Cosmos-Transfer2.5) |
| **Org** | NVIDIA |
| **Date** | 2026-02-26 (arXiv 2511.00062v2) |
| **Artifact** | `paper.pdf`, `paper.html`; github.com/nvidia-cosmos/cosmos-predict2.5, cosmos-transfer2.5 |
| **Disclosure level** | **A — full report; source code, pretrained checkpoints, and curated benchmarks released under NVIDIA Open Model License** |
| **Corpus** | **200M curated video clips** |
| **Successor** | **[Cosmos 3](../cosmos-3/dataprocess.md)** (2026-06) — see there for the full five-stage curation pipeline and SILA infrastructure |

## 1. Headline data fact

> *"Trained on **200M curated video clips** and refined with **reinforcement learning-based post-training**."*

The word "curated" is doing real work: the platform-level figure NVIDIA reports elsewhere is **20M hours of raw video reduced to ~100M clips** by the curation pipeline, processed by **NeMo Curator on Blackwell in ~14 days versus >3 years CPU-only**. The curation stack (5 stages: collect/preprocess → embed/dedup → categorize/filter → annotate → shard) is documented in detail in the Cosmos 3 report and analyzed in [that entry](../cosmos-3/dataprocess.md).

## 2. Why this entry matters for the survey — data *generation*, not just data *cleaning*

Cosmos-Predict2.5 unifies **Text2World, Image2World, and Video2World** generation in one flow-based model, and its stated purpose is explicitly to produce training data for other systems:

> *"These capabilities enable more reliable **synthetic data generation, policy evaluation, and closed-loop simulation** for robotics and autonomous systems."*

This is the supply side of the synthetic-data economy that several works in this survey consume:
- **GR00T N1** uses neural trajectory generation to take teleop from 88 h → 827 h
- **RoboCurate** verifies generated trajectories by simulator replay
- **LingBot-VA 2.0** scores generated human video for semantic preservation and physical plausibility
- **Hydra-0** trains and compares against Cosmos 2.5 backbones directly

## 3. Cosmos-Reason1 as a grounding component
> *"leverages **Cosmos-Reason1**, a Physical AI vision-language model, to provide **richer text grounding and finer control** of world simulation."*

A physics-aware VLM in the generation loop is the same architectural idea as the AI-judge in Cosmos 3's curation (Gemma-4 as auditor) and RoboCurate's attentive probe: **a learned physical-plausibility critic sitting beside the generator.** In the Cosmos family this critic appears at three points — data curation (Cosmos-Reason/Evaluator), generation grounding (Cosmos-Reason1), and quality assessment in the Physical AI Data Factory Blueprint.

## 4. Cosmos-Transfer2.5 — Sim2Real / Real2Real as a curation tool
> *"a **control-net style framework for Sim2Real and Real2Real world translation**. Despite being **3.5× smaller than Cosmos-Transfer1**, it delivers higher fidelity and robust long-horizon video generation."*

Domain translation is a *data-augmentation* primitive: it converts cheap simulation data into visually realistic training data, and re-renders real data into new visual domains. Compare Qwen-RobotManip's H2R compositing and RoboCurate's I2I/V2V augmentation — same function, different implementation.

## 5. Release posture
> *"we release **source code, pretrained checkpoints, and curated benchmarks** under the NVIDIA Open Model License… We hope these open resources **lower the barrier to adoption**."*

Releasing *curated benchmarks* alongside models is the practice the [VLA data survey](../vla-data-survey/dataprocess.md) identifies as necessary for comparability, and which most entries in this survey do not follow.

Models released at **2B and 14B** scales.

## 6. What to read where
| Question | Entry |
|---|---|
| How is the 200M-clip corpus actually filtered? | [Cosmos 3](../cosmos-3/dataprocess.md) — five stages, dedup at 20K clusters, DOVER/VTSS scores, ~100 artifact tags |
| How is the infrastructure built? | [Cosmos 3](../cosmos-3/dataprocess.md) — SILA, Lance columnar layer, LanceDB IVF_PQ |
| How is the generated data consumed as robot policy data? | [Cosmos Policy](../cosmos-policy/dataprocess.md) |

## 7. Transferable takeaways
1. **A world model is a data-generation instrument**, and its curation quality propagates into every downstream corpus built with it.
2. **Put a physics-aware critic next to the generator** (Cosmos-Reason1) as well as inside the curation pipeline.
3. **Domain translation (Sim2Real / Real2Real) is an augmentation primitive** that converts cheap data into usable data — and a 3.5× smaller model doing it better means the cost of that augmentation is falling.
4. **Release curated benchmarks with the model.** Comparability is a curation deliverable.
