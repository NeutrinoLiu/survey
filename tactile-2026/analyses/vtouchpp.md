# VTouch++ — A Multimodal Dataset with Vision-Based Tactile Enhancement for Bimanual Manipulation

**arXiv:2604.20444** · Humanoid Robot (Shanghai) Co. + Tongji + HIT (Hua, X. Li, Z. Yan, Y. Li, C. Zhang, Yongyao Li, Yufei Liu) · Apr 2026

**One line.** Organises a bimanual tactile dataset along **skill axes** rather than task labels — coordination pattern × atomic action × modality × temporal structure — so tasks can be recomposed analytically instead of segmented ambiguously.

## 1. What it collects

Synchronised **joint-level proprioception**, **multi-view RGB-D**, and **fingertip vision-based tactile** (GelSight-lineage), across **heterogeneous real platforms**:
- **Qingloong** — bipedal humanoid
- **Wheelloong M1** — wheeled humanoid
- **UMI-style mobile manipulators**

**380+ bimanual tasks** built from **100+ atomic action compositions**. All data is real-hardware execution, *"avoid[ing] sim-to-real artifacts."*

## 2. The skill-axis design

The distinctive contribution. Rather than a flat list of task labels:

> *"We structure demonstrations along fundamental axes including bimanual coordination patterns, atomic manipulation actions, sensory modalities, and temporal organization. Over 300 bimanual tasks are represented as compositions of atomic actions under diverse coordination and contact conditions, **enabling systematic recomposition and analysis without requiring ambiguous sub-trajectory segmentation**."*

The last clause is the practical payoff: sub-trajectory segmentation is a persistent source of label noise in long-horizon manipulation datasets, and a compositional task definition sidesteps it.

Their gap analysis of existing resources is a useful three-way summary: **human bimanual datasets** have scale and behavioural diversity but no robot proprioception, contact forces or embodiment constraints; **synthetic/simulation datasets** have precise state and force annotations but sim-to-real discrepancy; **real-robot datasets** are physically realistic but limited in scale, modalities, or task diversity.

## 3. Results

Benchmarked with **Action Chunking Transformers** and **diffusion policies with visual-tactile fusion**, plus a **cross-modal retrieval** evaluation over visual–tactile–pose representations.

The retrieval framework is trained end to end with learnable encoders and temperature parameters, outperforming baselines and *"validat[ing] that visual-tactile-pose representations learned via our framework capture meaningful cross-modal correspondences."*

Commendably, the authors flag the limit of their own strongest number: strong in-distribution action reconstruction *"serves primarily as an auxiliary validation of model capacity rather than evidence of generalization."*

## 4. Stated limitations

- Dual-arm platforms with fixed or wheeled bases; **mobile humanoid** embodiments would broaden generalisation evaluation.
- The benchmark evaluates **short-horizon skill retrieval**, not full task-level policy learning, despite the framework supporting long-horizon compositions.
- Cross-modal retrieval is **supervised with paired modality data**; self-supervised or unpaired regimes would scale better.

## 5. What it adds that the others don't

**Compositional task structure** as a dataset design principle, and **cross-embodiment collection across bipedal, wheeled and UMI-style platforms** in one resource. The skill-axis framing is the most transferable idea: it gives a controlled way to study *"the role of tactile feedback, coordination complexity, and generalization across task compositions"* — the kind of factorial analysis [[softvtbench]] achieves through matched rigid twins and [[taco-bench]] through matched sensors, here achieved through task decomposition.
