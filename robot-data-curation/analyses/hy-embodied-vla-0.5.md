# Hy-Embodied-0.5-VLA — companion artifact

**This entry is covered within the Tencent Hunyuan stream analysis:**
### → [HY-Embodied / RxBrain — Pretraining Data Curation & Cleaning Pipeline](../hy-embodied-0.5/dataprocess.md)

## Card

| Field | Value |
|---|---|
| **Work** | Hy-Embodied-0.5-VLA: From Vision-Language-Action Models to a Real-World Robot Learning Stack |
| **Org** | Tencent Hunyuan |
| **Date** | 2026-06 (arXiv 2606.14409) |
| **Artifact** | `paper.pdf`, `paper.html` |
| **Disclosure level** | **A — full report** |
| **Corpus** | *"more than **ten thousand hours** of high-quality data"* |

## Role in the stream

Tencent's embodied family splits three ways, each with a different data profile:

| Model | Role | Data |
|---|---|---|
| **Hy-Embodied-RxBrain-1.0** (~6.2B, Apache-2.0, 2026-07) | Reasoning engine — embodied QA/CoT, world-state prediction, subgoal planning | Taxonomy-driven mixture over **>100M samples** |
| **Hy-Embodied-VLM-1.0** | Perception at ~1/10 the compute | — |
| **Hy-Embodied-0.5-VLA** *(this entry)* | Action — the full real-world robot learning stack | **>10,000 hours** of high-quality data |

The VLA report is the *action* half of the story whose *understanding* half is documented in HY-Embodied-0.5 / RxBrain: the taxonomy-first curation (Action-Relevant State Understanding → Action–Transition Reasoning → Sequential and Adaptive Reasoning), the early-pretraining injection of 2D/3D grounding, depth and segmentation data, the mid-training blend with general-domain data, and the self-evolving rejection-sampling-SFT post-training loop.

It is tracked separately because it carries the stream's only published **hour-scale** figure for action data, and because "from VLA models to a real-world robot learning stack" is where the taxonomy-driven mixture is converted into deployable control.

See [HY-Embodied / RxBrain](../hy-embodied-0.5/dataprocess.md) for the full pipeline analysis.
