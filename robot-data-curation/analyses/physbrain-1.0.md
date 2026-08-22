# PhysBrain 1.0 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | PhysBrain 1.0 Technical Report |
| **Org** | DeepWisdom AI |
| **Date** | 2026-05 |
| **Artifact** | arXiv 2605.15298 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — full technical report; §2 is entirely the data engine** |
| **Corpus** | **~1,000 hours** of first-person human video → structured QA supervision |
| **Stance** | **"understanding first, action next."** Convert human video into *physical-commonsense QA*, not action labels. |

## 1. Thesis — the data engine as a compiler

PhysBrain's target is different from every other human-video pipeline here: it produces **question-answer supervision for a VLM**, which is then transferred into VLA policies. The governing metaphor is explicit:

> *"This design makes the data engine **closer to a compiler than to a caption generator**. Raw video is first parsed into an explicit physical record; the record is then augmented, checked, and finally rendered into QA supervision. **Each stage has a constrained input-output interface, so errors can be detected before they [propagate]**."*

And the rejected alternative is named:
> *"A naive answer would be to attach captions to video clips and ask the model to imitate those descriptions. We do not follow [this]"* — because captions capture *"high-level events while leaving out the physical structure needed for action generation, such as object geometry, contact progression, relative distance, reachability, or the order of sub-actions."*

## 2. Two design principles

1. **Supervision must be physically explicit.** Records describe *"not only what is visible, but also which objects are present, what physical attributes they have, how they are spatially arranged, how depth relations are formed, and how the scene changes under action."*
2. **Separate scene meta-information from model supervision.** Intermediate annotations are **structured/machine-readable** (source records for downstream generation); final training data are **natural-language QA pairs**. *"This separation lets PhysBrain 1.0 control the physical content of the data without reducing the model's training target to rigid JSON fields."*

This intermediate-representation discipline is the most transferable idea here: a machine-checkable middle layer that is *not* the training target.

## 3. Pipeline stages

```
raw ego video
  → CLIP FILTERING (visual quality + camera motion)
  → structured scene meta-information extraction
  → depth-aware spatial augmentation
  → QA generation
  → quality control & noise suppression
  → PhysBrain VLM training → transfer to VLA
```

### Clip filtering (pre-annotation gate)
> *"Before annotation, clips are filtered with both **visual-quality scores** and **camera-motion scores**. In practice, camera motion is estimated from **VGGT-derived camera parameters** and summarized as a motion score; segments with sufficient visual quality and **bounded camera shake** are retained, while low-quality or unstable clips are removed before meta-information extraction."*

Filtering on **camera stability** specifically is well-targeted: downstream depth and spatial-relation extraction fail exactly when egomotion is violent, so the filter is matched to the annotator's failure mode rather than to generic aesthetics.

### Structured scene meta-information
Four record families extracted from video:
| Record | Content |
|---|---|
| **Scene elements** | Which objects are present, their physical attributes |
| **Spatial dynamics** | How spatial relations evolve during manipulation |
| **Action execution** | Which actions are physically feasible; how local execution supports the broader task objective; order of sub-actions |
| **Depth-aware relations** | How depth relations are formed |

### Staged corpus construction
Deliberately incremental rather than one static build:
- **Stage 1**: egocentric sources — **Ego4D, BuildAI, EgoDex**. Clips segmented and converted into structured meta-information.
- **Stage 2**: re-annotation expanded to **EPIC**, **SEA-Small**, with *"a stronger emphasis on physical reasoning: the objective is no longer only to identify what action occurs, but to organize the clip into objects, physical [structure]."*

### Depth-aware spatial augmentation
A dedicated stage enriching records with depth-grounded relations — the signal most missing from 2D captions and most needed for reachability/contact reasoning.

### QA generation & the capability taxonomy
Records are rendered into *"diverse natural-language QA across **spatial, temporal, embodied, and general multimodal** capabilities."* Generating supervision from a structured record (rather than directly from video) is what makes the QA distribution controllable by construction.

### Quality control and noise suppression
A dedicated section (§2.7), consistent with the compiler framing — each stage's constrained interface makes errors detectable at the stage boundary rather than at the end.

## 4. Results
Trained on **only ~1,000 hours** of first-person human data, PhysBrain 1.0 reports SOTA across:
- **VLM side**: ERQA, PhysBench, OCRBench, RealWorldQA, TextVQA
- **VLA side**: SimplerEnv-WidowX, SimplerEnv-GoogleRobot, LIBERO, RoboCasa-GR1

with *"especially strong out-of-domain performance on SimplerEnv."*

## 5. Position in the survey
PhysBrain is the clearest instance of a **third route** for human video, alongside the other two:
| Route | Exemplars | Human video becomes… |
|---|---|---|
| Pseudo-action labels | EgoScale, ACE-Ego-0, DYNA-2, Qwen-RobotManip H2R | approximate robot actions |
| Video-prediction objective | DYNA-2 video tier, LingBot-VA, Hydra-0 | future-frame supervision |
| **Structured reasoning supervision** | **PhysBrain**, RoboBrain, RxBrain | **physical-commonsense QA** |

Its 1,000-hour footprint versus DYNA-2's 1,000,000 suggests the QA route extracts far more supervision per hour — but of a different, non-action kind.

## 6. What they do not do
- No action labels, no retargeting, no embodiment alignment — deliberately out of scope.
- Filtering thresholds (visual-quality, motion-score cutoffs) not published numerically.
- QA generation relies on model-generated records; error rates of the extraction stage are not quantified against ground truth.
- No data-scaling curve; the claim is efficiency at fixed small scale.

## 7. Transferable takeaways
1. **Build a structured intermediate representation that is not the training target.** It makes content controllable and errors checkable.
2. **Treat the pipeline as a compiler**: constrained input/output interfaces per stage, errors caught at stage boundaries.
3. **Filter on camera stability (VGGT motion score)** when downstream annotators are geometry-based.
4. **Match the filter to the annotator's failure mode**, not to generic quality notions.
5. **Human video can be worth far more per hour as reasoning supervision than as action supervision** — 1,000 h here versus 1,000,000 h for action-oriented pretraining.
