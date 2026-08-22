# Kairos — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Kairos: A Regret-Aware Native World-Action Model Stack for Physical AI |
| **Org** | Kairos Team |
| **Date** | 2026-07-07 (arXiv 2606.16533v3) |
| **Artifact** | `paper.pdf`, `paper.html`; github.com/kairos-agi/kairos; HF `kairos-agi`; ModelScope |
| **Disclosure level** | **A+ — every filter is named with its concrete implementing model.** §4 is an eight-subsection data chapter, including a section on its own limitations. |
| **Stance** | **"quality filtering is only the first layer… A visually clean clip is not necessarily control-informative."** |

## 1. The central distinction — quality vs. control-relevance

Kairos makes the sharpest conceptual contribution to curation in this survey:

> *"for Physical AI, **quality filtering is only the first layer. A visually clean clip is not necessarily control-informative.** Therefore, Kairos interprets data curation as a **two-level process: basic quality filtering followed by control-relevant event filtering**."*

Every other pipeline here optimizes for *quality* (sharpness, motion, aesthetics, label correctness). Kairos argues that quality is necessary but says nothing about whether a clip teaches you anything about **control** — and that the clips that do are systematically rare.

## 2. The Cross-Embodiment Data Curriculum — organized by intervention strength

> *"Instead of treating open-world videos, human demonstrations, and robot data as a **flat mixture**, Kairos organizes them into a **progression over intervention strength**."*

| Stage | Source | Intervention type | What it supplies |
|---|---|---|---|
| **I. Physical pretraining** | Open-world videos | **Passive observation** | *"broad environmental dynamics, object motion, scene evolution, and physical regularities without direct robot intervention"* |
| **II. Embodied pretraining** | Human behavioral data | **Intentional intervention** | *"task organization, goal-directed interaction, object manipulation strategies, and structured behavior patterns"* |
| **III. Regret-aware world-action training** | Robot interaction data | **Embodied intervention** | *"perception–action alignment, embodiment-specific constraints, actuation limits, execution errors, proprioception, and motor affordances"* |

The stated purpose: *"to move the model **from observation–action correlation toward action–outcome causation**."*

This is a genuinely different organizing axis from GR00T's data pyramid (quantity × embodiment-specificity) or Cosmos 3's quality ladder. **Intervention strength** is a causal-inference framing: passive video shows correlations, human data shows intent, robot data shows consequences. The section title says it explicitly — *"From Flat Data Scaling to a Staged Cross-Embodiment Curriculum."*

## 3. Level 1 — Basic quality filtering, with every tool named

The most reproducible filter specification in this survey. Each filter states its rationale, its implementing model, and its policy:

| Filter | Implementation | Rule | Rationale |
|---|---|---|---|
| **Aesthetic Score** | CLIP backbone + MLP head | Drop below threshold | *"Low-aesthetic videos may contain simplistic scenes, poor color distribution, or insufficient visual structure"* |
| **Motion Score** | **RAFT** optical flow, magnitude aggregated globally | Drop clips with **excessively low OR high** motion | *"Static videos contain insufficient temporal information, while videos with extreme flickering or unstable motion can harm temporal consistency"* |
| **AIGC Score** | **ViT-Large discriminator** trained on a proprietary synthetic-video dataset | Exclude above AIGC threshold *"in relevant training stages"* | *"low-quality synthetic content can introduce artifacts, unrealistic dynamics, or **misleading physical patterns**"* |
| **NSFW Score** | **Falconsai** (open-source) | Filter unsafe content from crawled data | Safety |
| **Blurriness Score** | **Laplacian operator** | **Remove *or downweight*** below sharpness threshold | *"Sharpness affects the model's ability to learn object boundaries, contact points, motion cues, and spatial structure"* |
| **Human Motion Score** | **YOLOX** detection + **ByteTrack** tracking → normalized pixel velocity per person | Select clips with meaningful human motion | *"a dataset dominated by static human clips may not teach useful body, hand, or manipulation dynamics"* |
| **OCR Score** | **DBNet** text-region detection → proportion of text regions | Exclude text-heavy clips **in early phases**; *"text-rich clips may be selectively used in later stages if text generation or GUI-like tasks are relevant"* | *"text rendering can interfere with convergence"* |
| **Deduplication** | **CLIP** video embeddings + persistent embedding pool; pairwise similarity vs. history | Above threshold, **retain only the higher-resolution/higher-quality version** | *"Redundant clips increase storage and compute cost without adding new information"* |

Three details worth copying:
- **Two-sided motion thresholds.** Nearly every other pipeline filters only *static* clips; Kairos also rejects excessive motion, because flickering destroys temporal-state learning.
- **AIGC detection as a filter.** As of 2026, crawled video is substantially AI-generated, and synthetic video with wrong physics is *actively harmful* to a world model. No other entry in this survey filters for this.
- **Stage-dependent filters.** OCR and AIGC thresholds vary by training phase — a filter is not a global property of the corpus but a function of what the current stage needs.

And the unifying rationale, tying quality back to control:
> *"these filters also remove noise that can obscure physical and action-relevant variables: **blurry videos make contact points harder to infer; unstable flickering weakens temporal state learning; corrupted synthetic videos may teach incorrect dynamics**."*

## 4. Level 2 — Control-relevant event filtering (the roadmap, honestly labelled as unimplemented)

Kairos defines **control information density (CID)** and enumerates six event families that carry it. It also states plainly that this level is not yet built:

> *"The current pipeline **does not yet compute CID directly**; future iterations should explicitly target:"*

| Event family | Examples |
|---|---|
| **Near-boundary failures** | marginal grasp failure, near slip that becomes a drop, near collision that becomes contact, unstable stack collapse, partial task failure |
| **Recovery events** | regrasping, repositioning, replanning, human correction, retry behavior, post-failure continuation |
| **Near-boundary successes** | marginal grasps that remain stable, near slips that are corrected, near collisions avoided, partial successes completed through correction |
| **Contact transitions** | first contact, loss of contact, grasp closure, slip, collision, support change |
| **Safety and anomaly events** | human proximity, sharp-object contact, excessive force, irreversible state change |
| **Long-horizon dependencies** | delayed failure, multi-step dependency, hidden object state, task-progress change |

The argument for why these matter is the best statement in this survey of *why success-only corpora fail*:

> *"These events are **rare compared with ordinary successful clips**, but they are highly valuable when they are close to the task manifold and diagnostically interpretable. **A model that never sees near-boundary failures may generate clean success futures but remain unable to predict where deployment will break. A model that never sees recovery may fail to plan after mistakes. A model that never sees near-boundary successes may not learn which small corrections or safety margins preserve successful execution.**"*

This provides the theoretical justification for what π₀.₇ (mistake labels), AGIBOT (retained error-recovery trajectories), RECAP (advantage-graded autonomous rollouts), and ACE-Brain-0.5 (oracle recovery actions) each do empirically. **Near-boundary successes** in particular is a category no other work names — the clips where things almost went wrong but didn't are the ones that teach safety margins.

Future data sources proposed for computing CID: *"explicit detectors, human-in-the-loop annotation, simulation labels, robot rollout logs, and tactile–force signals."*

## 5. Structured tagging for control-relevant sampling

> *"Each video is assigned to **exactly one domain** to ensure **unambiguous data partitioning**."*

| Tag | Sub-tags | Purpose |
|---|---|---|
| **Human** | scenes / actions / occupation / gender / age / context / **face blur** / body motion | Human behavior patterns |
| **Robot** | scenes / actions | Robot–environment interaction mechanisms |
| **Physics** | principles | Physical-rule modeling |
| **General** | content type / scene / animal | — |

Mutually exclusive domain assignment makes mixture proportions exactly computable — the same motivation as EgoVerse's SQL metadata layer and HumanNet's multi-axis index. Note **face blur** as a tracked sub-tag, i.e. privacy state is part of the index.

## 6. Data-chapter structure (§4) — worth noting as a template
4.1 Data Collection: Multi-Source Experience Acquisition · 4.2 Data Curation · 4.3 Tagging: Structured Indexing for Control-Relevant Sampling · … · 4.6 Data Engineering Infrastructure for Scalable Control-Information Processing · **4.7 Limitations and Future Data Directions** · 4.8 Summary: A Data Engine for Control-Sufficient World Modeling.

A dedicated *Limitations* subsection inside the data chapter is close to unique in this survey, and Kairos is candid elsewhere too: *"Direct validation of real-robot closed-loop regret reduction… remains an important direction for future Kairos development"* — i.e. the regret framing is supported by proxy evidence, not closed-loop robot results.

## 7. What they do not do
- Numeric thresholds are not published (tools and policies are; constants are not).
- **CID / control-relevant filtering is specified but not implemented** — stated openly.
- No corpus size or per-source hour counts.
- No fitted data-scaling law; results are benchmark comparisons.

## 8. Transferable takeaways
1. **Separate quality from control-relevance.** Clean ≠ informative. This is the single most useful reframing in the survey.
2. **Organize the curriculum by intervention strength** — passive video → intentional human behavior → embodied robot action — to move from correlation toward causation.
3. **Name your filters and their implementations.** RAFT / CLIP+MLP / ViT-L AIGC discriminator / Falconsai / Laplacian / YOLOX+ByteTrack / DBNet / CLIP-dedup is a directly reusable stack.
4. **Filter motion on both sides** — too little *and* too much.
5. **Filter AI-generated video out of world-model training data.** Wrong physics is worse than no data, and crawled video is increasingly synthetic.
6. **Make filters stage-dependent**, not global properties of the corpus.
7. **Collect near-boundary successes, not just failures.** Nobody else names this category, and it is where safety margins are learned.
8. **Assign exactly one domain tag per sample** so mixture proportions are computable rather than estimated.
9. **Write a limitations subsection inside your data chapter.** Kairos states which of its own filters do not exist yet.
