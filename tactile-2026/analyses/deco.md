# DECO — Decoupled Multimodal Diffusion Transformer for Bimanual Dexterous Manipulation with a Plugin Tactile Adapter

**arXiv:2602.05513** (v4, Aug 2026) · **ICML 2026** · XYZ Embodied AI + BAAI + TUM + UCAS + Tsinghua + Nanjing (X. Li, Sun, L. Zhang, B. Huang, Peng, Meng, H. Jiang, S. Xie, Yao, Knoll, Bing, X. Wang, Z. Sun) · Feb 2026 · [code](https://github.com/BAAI-Humanoid/DECO) · [dataset](https://huggingface.co/datasets/BAAI-Humanoid/DECO-50)

**One line.** Assigns each modality its **own injection pathway** into a diffusion transformer — and then proves the assignment is not arbitrary by swapping two of them, which drops one task stage from **29/40 to 0/40**.

## 1. What "tactile" means here

Tactile from a **dexterous hand**, injected by **cross-attention** — deliberately different from how vision and proprioception enter.

The design premise: *"in such visuo-tactile policies, modalities are typically fused in a coupled manner with equal importance, which may fail to fully exploit the distinct roles of vision, proprioception, and touch — especially when vision changes rapidly with an active camera and tactile signals remain sparse during manipulation."*

## 2. Data curriculum — DECO-50

| | |
|---|---|
| Duration | **50+ hours** |
| Frames | **~5 M** |
| Successful trajectories | **8,000** |
| Scenarios | 4 |
| Sub-tasks | 28 |
| Collection | teleoperation on real dual-arm robots, with **active vision** |

Notably, the dataset deliberately *"includes both contact-rich tasks that benefit from tactile and tasks where visual and proprioceptive information suffice"* — a design choice that lets the paper answer when tactile is *not* worth collecting.

Two-stage training: (1) vision–action policy on images, proprioception, task conditions; (2) **freeze it**, add tactile via a lightweight adapter and cross-attention.

## 3. Model — three pathways

| Modality | Injection mechanism | Rationale |
|---|---|---|
| **Image + action tokens** | joint self-attention | dense, spatially structured, must interact |
| **Proprioception + optional conditions** | **AdaLN** | global, low-dimensional conditioning |
| **Tactile** | **cross-attention** | sparse, needs selective attention |

Plus a **LoRA-based plugin adapter** for parameter-efficient fine-tuning of the pretrained policy.

## 4. How tactile enters the model

Cross-attention with LoRA. The parameter accounting:

| Variant | Total params (M) | **Trainable (M)** |
|---|---|---|
| DECO (vision only) | 83.05 | 83.05 |
| DECO.cs (coupled/simple merge) | 84.14 | 84.14 |
| DECO.ds (tactile from scratch) | 89.41 | 89.41 |
| **DECO.p (plugin adapter)** | 91.02 | **7.97** |

DECO.p trains only the **tactile encoder, tactile cross-attention K/V projections, and LoRA** — **7.97M parameters, under 10%** — and reaches performance comparable to DECO.ds trained from scratch.

## 5. Results

**Main:** **72.25%** average success across all tasks, **+21%** over baseline, evaluated with over **2,000 real robot rollouts**. The tactile adapter adds **+10.25%** average and **+20% on complex contact-rich tasks**.

**The pathway-assignment ablation is the paper's best experiment.** Swap the two auxiliary modalities — proprioception via cross-attention, tactile via AdaLN — keeping everything else identical:

| Object | Stage | DECO.ds | DECO.ds **(permuted)** |
|---|---|---|---|
| 00296-2x | 1 | 19/20 | 15/20 |
| | 2 | 17/20 | 6/20 |
| | 3 | 14/20 | **0/20** |
| IF (hose/nozzle) | 1 | 18/20 | 18/20 |
| | 2 | 17/20 | 11/20 |
| | 3 | 15/20 | **0/20** |
| **Total** | 3 | **29/40** | **0/40** |

Complete failure on the final stage of both objects. The mechanistic explanation is specific and credible: injecting **proprioception via cross-attention treats each joint as a sequence element**, causing the model to over-emphasise proprioceptive signals — real-world rollouts show visible **jitter**. Conversely **AdaLN captures global conditioning**, which is too coarse for the fine-grained tactile sensitivity assembly needs.

This is the strongest available evidence that *which* fusion mechanism a modality gets is not an implementation detail. It complements [[forcevla2]]'s injection-*point* ablation (VLM pathway = 5%) and [[at-vla]]'s direct-injection collapse: three papers, three axes — where, how, and in what format — all showing the same fragility.

**When does tactile matter?** The task breakdown is refreshingly negative where warranted: tactile benefits **waste disposal and assembly** (contact detection, force monitoring, visual occlusion), while **pick-and-place and material sorting** achieve strong performance from vision and proprioception alone — *"suggesting that tactile collection may not be justified for such scenarios."*

**Qualitative mechanism** on the interference-fit hose/nozzle pair: without tactile, DECO is prone to **misalignment, under-pressing** (failing to pass the retention ring), or **over-pressing** (bottoming out and stressing the hose). With tactile, it senses whether the hose has slid past the ring and uses force cues as a completion signal — improving Stages 2 and 3 specifically. A clean illustration of touch supplying a *termination condition* that vision cannot.

**Stated limitations.** Single hardware platform, one dexterous hand, one tactile sensor type; no temporal modelling or memory for long-horizon contact-rich manipulation; the adapter is untested on larger pretrained VLAs.

## 6. What it adds that the others don't

The **permuted-pathway experiment**. Nothing else in this survey tests whether the modality→mechanism assignment could be swapped, and the answer — total failure at the precision stage — establishes that decoupling is doing real work rather than adding capacity. The **7.97M-parameter plugin adapter** matching from-scratch training is also the most parameter-efficient tactile integration here, and DECO-50's inclusion of tasks where tactile *doesn't* help makes it one of the few datasets that can support a negative result.
