# EgoVerse — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | EgoVerse: An Egocentric Human Dataset for Robot Learning from Around the World |
| **Org** | Consortium — Georgia Tech, Stanford, UC San Diego, ETH Zurich, MIT, Meta Reality Labs + industry partners |
| **Date** | 2026-04 |
| **Artifact** | arXiv 2604.07607 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — full paper; the pipeline *is* the contribution** |
| **Corpus** | **1,362 h**, 1,965 tasks, 240 scenes, **2,087 demonstrators** |
| **Stance** | **Curation as infrastructure.** A *living* dataset with a nightly validation pipeline, not a static release. |

## 1. The thesis — "bounded diversity" and a living dataset

EgoVerse names the failure mode of prior human-video corpora explicitly: Ego4D, EPIC-KITCHENS, Something-Something V2, HOI4D, EgoExo4D *"are not designed for robot learning… include tasks beyond current robot capabilities, lack manipulation-relevant annotations… and contain unstructured activities that are difficult to translate into executable robot demonstrations."*

Its answer is **bounded diversity**: constrain the *task* distribution to what a bimanual mobile manipulator could plausibly execute, while maximizing variation in scene, object, and demonstrator. This is curation applied at **collection-specification time** rather than post-hoc filtering — the cheapest possible place to do it.

Second thesis: prior datasets are *"one-off, static releases… making further scaling difficult."* EgoVerse is built as a continuously-ingesting platform.

## 2. Two-stream architecture — a deliberate quality/scale split

| Stream | Purpose | Collection | Annotation density |
|---|---|---|---|
| **EgoVerse-A** (academic) | Reproducible, controlled study | Standardized protocol mirrored across labs | **Lightweight per-episode**: task description, scene ID, primary manipulated objects, demonstrator metadata |
| **EgoVerse-I** (industry) | Scale, diversity, richness | In-the-wild by industry partners | **Dense**: fine-grained (1–2 s) language descriptions, active-hand indicators, static-vs-mobile manipulation flags, contextual tags |

EgoVerse-I is *"the largest action-labeled egocentric human dataset"* — ~1,400 h / ~2,000 tasks / 240 scenes / 2,087 demonstrators. Note that Qwen-RobotManip consumes **only the industry portion (~954 h)** for pretraining.

## 3. Collection-time quality constraints (the primary filter)

**EgoVerse-A protocol**, organized around **dataset units**:
- One unit ≈ **5 minutes of recording → 5–10 demonstrations** per task, common instruction format across all sites.
- **Enforced quality constraints: hands and manipulated objects must remain visible.**
- Logged metadata for traceability: demonstrator identity, scene, object set.
- **Six shared Flagship Tasks** across all labs to hold task semantics constant while other axes vary (e.g. `object-in-container`: pick, place into container, dump, repeat continuously for 40 s with randomized placement).
- Workspace bounded to roughly **40 cm × 60 cm**; 8–12 scenes per lab per flagship task, 1–10 units per scene.

**EgoVerse-I quality control:**
- Demonstrators **instructed to keep hands visible**.
- Task distributions deliberately **manipulation-heavy**.
- **Manual quality control to retain only manipulation-dense segments** — an explicit human-in-the-loop segment-level filter. This is the one place EgoVerse pays for cleaning with labor rather than heuristics.

## 4. Annotation pipeline

Per frame, every episode carries at minimum: **egocentric video + 3D hand keypoints + camera pose**.
- **3D hand pose**: 21 keypoints per hand, both hands, in the camera frame.
- **6-DoF head pose** from visual–inertial SLAM.
- **Academic partners** use Project Aria's Machine Perception Service (MPS) for tracking and egomotion.
- **Industry partners** combine partner SLAM + model-based pose estimation + post-processing *"to ensure consistent trajectories."*

Heterogeneous perception backends are reconciled by a shared *output* schema rather than a shared *method* — a pragmatic federation choice.

## 5. EgoDB — the ingestion and validation system

This is the piece most other works lack.

```
distributed contributors → S3-backed storage
   → conversion to a unified training-ready format
   → NIGHTLY PIPELINE: standardized preprocessing, VALIDATION, indexing
   → episode metadata registered in a centralized SQL database
   → structured queries over task / embodiment / scene / source / annotation type
   → web viewer for browsing, annotation inspection, growth tracking
   → config-file sync of FILTERED SUBSETS for local reproducible training
```

Key properties:
- **Validation runs continuously (nightly), not once at release.** New contributions cannot silently degrade the corpus.
- **Filtering is a first-class query surface** — subsets are declared in config files, so a training mixture is reproducible and auditable by construction.
- Metadata in SQL makes diversity *measurable* (how many scenes? how many demonstrators?) rather than asserted.

## 6. Deliberately controlled scaling axes

EgoVerse-A is collected against a **structured assignment matrix** over a fixed pool of **16 demonstrators × 16 scenes** (3.75 min to 2 h per demonstrator–scene pair, ≥1 h per scene), to make three axes independently manipulable:

1. **Single-scene demonstrator scaling** — fixed 2 h budget, demonstrators 1 → 16.
2. **Multi-scene interaction effects** — scenes 1–8, demonstrators 4 → 12, fixed 8 h budget.
3. **Scene diversity scaling** — scenes 1–16, fixed demonstrator pool, decoupling scene diversity from data quantity.

Designing the *collection* so that diversity axes can be ablated is a genuinely distinctive contribution: most corpora can only report total hours.

## 7. Scaling evidence — and the important negative result

- Co-training with EgoVerse improves robot performance, with **up to 30% relative gains** across robots and tasks.
- **Scaling is conditional, not automatic.** *"neither 8 h of diverse EgoVerse-A data nor domain-aligned human data alone is sufficient to drive significant performance gains in ID or OOD settings, we observe positive scaling when domain-aligned data 'anchors' the learning process."* As little as **2 h of domain-aligned data unlocks transfer from otherwise inert diverse data.**
- **Diversity beats density past a threshold**: *"Once data quantity reaches a moderate level, increasing data density yields diminishing returns, whereas expanding scene coverage provides measurable gains."*

The headline conclusion — *"robot performance improves with increased human data, but effective scaling depends on alignment between human data and robot learning objectives"* — is the strongest published caution against treating raw hours as the scaling variable.

## 8. What they do not do
- No automated episode rejection cascade (EgoVerse-I quality control is manual).
- No dedup across contributors.
- No retargeting to robot action spaces in the release itself — hand/head poses serve as *proxies* for end-effector motion, and consumers (e.g. Qwen-RobotManip) do their own retargeting.
- Perception backends differ between academic and industry streams; no cross-stream calibration study is reported.

## 9. Transferable takeaways
1. **Specify quality at collection time.** "Keep hands and objects visible" prevents a whole class of unusable episodes for free.
2. **Bounded diversity** — constrain the task axis to robot-feasible behaviors, maximize every other axis.
3. **Nightly validation + SQL metadata + config-declared subsets** is a reproducibility pattern the rest of the field mostly lacks.
4. **Design collection as an experiment.** The 16×16 demonstrator×scene assignment matrix makes diversity ablations possible at all.
5. **A small aligned anchor is what makes diverse data usable** — 2 h of domain-aligned data unlocking 8 h of diverse data mirrors EgoScale's 829 h EgoDex anchor and Ψ₀'s post-training stage. Three independent works converge on this.
