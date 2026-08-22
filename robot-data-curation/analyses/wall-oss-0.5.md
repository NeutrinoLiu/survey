# Wall-OSS-0.5 / WALL-WM (X Square Robot) — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Works** | **WALL-OSS** (2025-09) → **Wall-OSS-0.5: A Deployment-Ready VLA with Gradient-Bridged Pretraining** (2026-05) · **WALL-WM: Carving World Action Modeling at the Event Joints** (arXiv 2606.01955, 2026-05) · Wall-X 1.1.0 stack (2026-06) |
| **Org** | X Square Robot (自变量机器人) |
| **Artifacts** | `README.md` (github.com/X-Square-Robot/wall-x), PR Newswire release |
| **Disclosure level** | **B — open-source code and weights + press release.** Data mixture is described in prose; no hour counts, thresholds, or rejection rates. |
| **Corpus** | Three-source mixture including a **90M-sample multimodal corpus** |
| **Stance** | **Single-stage training on a bridged mixture** — no separate pretrain/finetune split. |

## 1. Mission statement — data as the compressed object

> *"We are building embodied foundation models to capture and compress the world's **most valuable data: continuous, high-fidelity physical interaction**. By creating a **direct feedback loop between model decisions and the body's lived experience**, we enable generalizable intelligence."*

## 2. The three-source mixture

Wall-OSS-0.5 is *"trained in a **single stage** on a three-source mixture"*:

| Source | Note |
|---|---|
| **Self-collected manipulation data** | From X Square's own collection stack |
| **Curated open-source multi-embodiment trajectories** | Public corpora, curated |
| **90M-sample multimodal corpus** | *"includes **embodied bridge samples synthesized from action trajectories**"* |

The distinctive element is the third: **embodied bridge samples synthesized from action trajectories**. Rather than co-training on generic VL data to preserve semantics (Qwen-RobotManip's 28M mixture, GR-3's VL corpus), X Square *derives* multimodal training samples **from the action data itself**, creating supervision that natively spans the semantic and motor domains. This is what "**Gradient-Bridged Pretraining**" names: the bridge samples carry gradient signal between the language/vision objective and the action objective, so the two do not have to be balanced against each other as competing losses.

**Single-stage training is itself a data claim.** The stated result — *"bringing pretrained VLA performance closer to post-training levels"* — is that a well-bridged mixture removes the need for a separate adaptation phase, in contrast to Galaxea G0's three-stage curriculum, GR00T's pyramid co-training, and Ψ₀'s sequential EgoDex→Humanoid-Everyday schedule.

## 3. Collection infrastructure

X Square has developed *"advanced data-capture tools, including **teleoperation, exoskeletons, and the Universal Manipulation Interface (UMI)**, and has established a data pipeline to generate high-quality data at scale."*

Three capture modalities with different cost/fidelity profiles — exoskeletons and UMI grippers in particular are the "hardware-assisted collection" category from the [VLA data survey](../vla-data-survey/dataprocess.md) taxonomy, and UMI-style data appears as a source in LingBot-VA 2.0 and Hydra-0 (via Deform360) as well.

WALL-A, the flagship end-to-end model, has a *"shared-attention, task-routed architecture [that] trains on **real robot data plus augmented generative video**."*

## 4. WALL-WM — event-boundary segmentation as the data primitive

> **WALL-WM: Carving World Action Modeling at the Event Joints** — *"a World Action Model that couples **future-video imagination with action prediction at semantic event boundaries**."*

The phrase *"carving at the event joints"* is a curation principle: rather than chunking trajectories at fixed intervals, segment them where the **semantics** change. Compare:
- **HumanNet** — scene splitting at visual changes, *"so that unrelated activities are not merged into a single sample"*
- **Hydra-0** — fixed non-overlapping 81-frame windows
- **ACE-Ego-0** — time-aligned chunking by physical timestamp
- **ABot-M0** — subtask decomposition from separate annotation files

WALL-WM argues the segment boundary should be a *semantic event joint*, which makes each training sample a coherent unit of prediction. Where you cut the trajectory determines what the model can learn about transitions.

## 5. Release posture
Open weights (`x-square-robot/wall-oss-flow` on HF) and an open training/inference stack (LeRobot data preparation, flow-matching and FAST action branches, serving/evaluation runtime, DMuon training support, install-time CUDA operator builds). Practically, the **LeRobot data-preparation path is the reusable artifact** — it is the format most 2026 corpora converge on (ABot-M0 standardizes everything to LeRobot v2).

## 6. What is not disclosed
- No hour counts for the self-collected or open-source components.
- No filtering criteria, quality gates, or rejection rates.
- The synthesis procedure for the "embodied bridge samples" is not detailed in the public artifacts.
- No mixture proportions among the three sources.
- No fitted scaling curve.

## 7. Transferable takeaways
1. **Synthesize bridge samples from your action data** rather than importing generic VL data — supervision that natively spans both domains removes the need to balance competing losses.
2. **Segment at semantic event boundaries**, not at fixed intervals. Where you cut determines what transitions the model can learn.
3. **Multiple capture modalities** (teleop / exoskeleton / UMI) span the fidelity-cost curve that the VLA data survey identifies as fundamental.
4. **Single-stage training is a testable claim about mixture quality** — if the mixture is well-bridged, the pretrain/finetune boundary can dissolve.
5. **Standardize on LeRobot format.** It is the de facto interchange format across the 2026 corpora in this survey.
