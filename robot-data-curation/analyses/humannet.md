# HumanNet — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | HumanNet: Scaling Human-centric Video Learning to One Million Hours |
| **Org** | DAGroup, Peking University |
| **Date** | 2026-05 |
| **Artifact** | arXiv 2605.06747 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — the curation pipeline is the paper's stated primary contribution** |
| **Corpus** | **1,000,000 hours**, 967K videos, first-person + third-person |
| **Stance** | *"treating curation, viewpoint diversity, and annotation taxonomy as core scientific contributions rather than bookkeeping."* |

## 1. Thesis

HumanNet is the most explicit statement in this survey that **data curation is the research object, not the plumbing**:

> *"HumanNet introduces a systematic data curation paradigm for embodied learning, where human-centric filtering, temporal structuring, viewpoint diversity, and annotation enrichment are treated as first-class design principles."*

It also gives the sharpest **inclusion definition** in the field — the scope is defined by what counts as physically grounded behavior, and the exclusion clause is written down:

> qualifying content = *manipulating objects, using tools, navigating task-relevant space, assembling/disassembling, operating appliances or interfaces, transporting objects, coordinating with other people, or executing multi-step procedures with visible state changes.*
>
> *"This definition intentionally excludes large volumes of passive or weakly grounded video in which human motion is incidental, the activity is not temporally coherent, or the recording lacks useful visual evidence for action, motion, or interaction."*

Four design principles: **Scale · Viewpoint diversity · Physical structure preservation · Pretraining readiness.**

## 2. The three-stage pipeline

The staging is deliberate: *"This staged design cleanly separates source acquisition from clip-level cleaning and from supervision generation, so that each stage can be audited, extended, or rerun independently as the corpus scales."*

### Stage 1 — Data collection (acquisition + source-level filtering)
```
seed keywords → keyword expansion → keyword-based crawling & cleaning
              → channel-level crawling → integration of existing sources
              → unified KEYWORD REPOSITORY
                   ↓ drives retrieval over
video-platform search · general web search · direct crawl · open-source datasets · self-collection
                   ↓
             unified pool of mixed videos
```
- **Channel-level and source-level filtering** removes off-topic, low-quality, or *passively observational* sources — cheap rejection at the granularity of a whole channel rather than a clip.
- Duplicate source entries and obviously unusable recordings pruned **before** downstream processing.
- **Self-collection stream** exists specifically to cover *"underrepresented activities, viewpoints, and scenes that are difficult to source reliably from public platforms"* — targeted collection as a long-tail repair mechanism.
- Output splits into an **ego-video URL pool** and third-person material *"retained when human motion and activity remain visually central."*

### Stage 2 — Data processing (clip-level cleaning)
Five ordered operations:

| Operation | Purpose |
|---|---|
| **De-duplication & normalization** | Remove near-identical copies; unify frame rate, resolution, container format |
| **Content filtering** | Retain clips with meaningful human action and observable motion |
| **Quality filtering** | **Discard severe motion blur, heavy occlusion, static framing, and other defects that undermine learning** |
| **Scene splitting** | Segment long videos at visual changes *"so that unrelated activities are not merged into a single sample"* |
| **Video clipping** | Emit fixed-granularity segments |

Note this is one of the few pipelines in the survey that runs **explicit near-duplicate detection** — a standard step in LLM/CLIP curation that most robotics corpora omit.

### Stage 3 — Annotation (supervision generation)

| Module | Output | Gate |
|---|---|---|
| 3D hand & body pose detection | Fine-grained motion structure | — |
| **Monocular SLAM** | Camera trajectory for ego clips | only for clips *"that satisfy stability and parallax requirements"* |
| **Retargeting to a unified humanoid skeleton** | Robot-ready motion | **clip marked robot-ready iff retargeting error < 15 mm AND valid-frame coverage > 60%** |
| LLM-assisted captioning | Video captions, motion descriptions, activity classifications | normalized against any inherited narrations/metadata |

The **15 mm / 60%** gate is the most concrete quantitative admission criterion published by any work in this survey. It also implies a **tiered corpus**: the full 1M hours for representation learning, a smaller certified "robot-ready" subset for action supervision.

Annotation is **multi-label, not mutually exclusive** — clips routinely combine manipulation, tool use, transport, locomotion, and multi-person coordination.

### Stage 4 (release gate) — Privacy, safety, licensing
*"Privacy-sensitive content, unsafe material, and license constraints are reviewed within the same release pipeline, since both first-person and third-person recordings can contain identifiable people, private spaces, documents, screens, or proprietary workflows."*
This is the only work in the survey that treats privacy review as a pipeline stage rather than an afterthought — notable given that egocentric home video is the dominant new data source across the whole field.

## 3. Indexing instead of homogenizing

A distinctive architectural choice: heterogeneity is **indexed, not removed**.

> *"Rather than treating this heterogeneity as noise, we index the corpus through a small set of factors that determine its value."*

Multi-axis taxonomy over: **source type · viewpoint · task structure · environment · interaction style · motion category · metadata availability.** Metadata provenance is tracked separately — some sources ship narrations/timestamps/task descriptions, others get pseudo-labels (hand tracks, body pose, motion categories, contact estimates, scene tags, captions, procedural boundaries).

> *"This structure supports flexible training mixtures without forcing all sources into a single annotation regime."*

The payoff is a corpus that is **stratified by physical quality**: after quality filtering the pose-score distribution concentrates at the high-confidence end; motion-score and motion-length distributions are heavy-tailed but bounded. *"High-confidence, well-segmented subsets concentrate the supervision needed for grounding, whereas the heavier-tailed regions supply the scale needed for long-tail behaviors."* Consumers select the slice matching their task — **mixed-supervision training** rather than one global mixture.

## 4. Two viewpoint-specific bridges to robots
- **Exocentric** → full-body motion → **retargeting** → robot motion
- **Egocentric** → paired hand pose → manipulation transfer

Most competing corpora commit to ego only; HumanNet's argument for keeping both is that *"the former captures execution-centered cues, while the latter captures full-body motion, scene context, and interactions among people and objects."*

## 5. Scaling / validation evidence

A **controlled substitution study** under a fixed LingBot-VLA architecture and fixed downstream corpus (100 tasks × 20 episodes = 34 h robot interaction), varying **only the pretraining source**:

| Pretraining source | Result |
|---|---|
| Qwen VLM (no embodied pretraining) | baseline |
| Qwen + **100 h real-robot CoBot data** | reference |
| Qwen + **1,000 h HumanNet egocentric video** | **on par with, and on several task groups below, the 100 h real-robot model** |
| LingBot (Qwen + **20,000 h real-robot**) | upper reference |

Claimed exchange rate: **~1,000 h of curated human video ≈ 100 h of real-robot data**, i.e. roughly **10:1**, and *"substantially closes"* the gap to the 20,000 h robot-pretrained model. That the study holds architecture and downstream data fixed makes it one of the cleaner source-substitution measurements available.

Rationale for 1M hours specifically: the activity-category distribution has *"a pronounced long tail"* — behaviors like folding deformable objects, handling reflective containers, or operating unfamiliar appliances *"appear often enough to contribute to representation learning, whereas at smaller scales they would be easily underrepresented."* Scale is justified as **long-tail coverage**, not as raw volume.

## 6. What they do not do
- No robot action labels in the general corpus — only the certified robot-ready subset carries retargeted kinematics.
- Validation is single-architecture and at 1,000 h; no fitted scaling curve across the full 1M h (contrast EgoScale).
- Retargeting quality gate applies to the humanoid skeleton only; no per-embodiment reachability check.
- Pose/SLAM/caption modules are pseudo-labelers; their error rates are characterized distributionally, not audited against ground truth.

## 7. Transferable takeaways
1. **Write down the exclusion clause.** HumanNet's explicit definition of non-qualifying video ("motion is incidental… not temporally coherent… lacks visual evidence") is reusable as a spec.
2. **Filter at the coarsest granularity that works** — channel/source-level rejection before clip processing saves orders of magnitude of compute.
3. **Publish a numeric admission gate** (< 15 mm retargeting error, > 60% valid-frame coverage). This makes "robot-ready" auditable.
4. **Scene splitting at visual changes** prevents the silent corruption of merging unrelated activities into one training sample.
5. **Index heterogeneity rather than erasing it**, and expose the quality strata so downstream users can pick their own operating point.
6. **Make privacy/licensing a pipeline stage**, not a release checkbox.
