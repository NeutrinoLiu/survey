# Cosmos 3 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Cosmos 3: Omnimodal World Models for Physical AI |
| **Org** | NVIDIA (Cosmos Lab) |
| **Date** | 2026-06-22 |
| **Artifact** | research.nvidia.com/labs/cosmos-lab/cosmos3/technical-report.pdf (`paper.pdf`, ~74K words) |
| **Disclosure level** | **A+ — the most complete data-infrastructure disclosure in this survey.** Named models, numeric thresholds, retention rates, cluster counts, index configurations. |
| **Corpus** | Pre-training: **767M images + 347.7M video clips**, processed from **7.8B raw images + 3B raw source videos** |
| **Stance** | **Curation is production infrastructure.** *"an iterative enrichment process rather than a single offline preprocessing pass."* |

## 1. Headline yield

| | Raw | Retained | **Yield** |
|---|---:|---:|---:|
| Images | 7.8B | 767M | **9.8%** |
| Videos | 3B source videos | 347.7M clips | — |

Roughly **90% of raw images are discarded.** The pipeline's own framing: *"the pipeline is both low-yield and highly iterative: only a small fraction of raw candidates ultimately survive into training, meaning that inefficient scans, copies, or model inference are disproportionately spent on samples that are later discarded."* Low yield is treated as an **engineering constraint that shapes the architecture**, not an incidental fact.

Retained-corpus composition: 720p = 26.8% of images / 36.4% of videos; 480p = 26.0% / 30.8%; ≥1080p = 25.2% / 12.2%. Dominant aspect ratio 16:9 (52.0% of images, **97.3%** of videos); 1:1 is 25.2% of images.

## 2. The five-stage image/video pipeline

> *"(1) collecting raw data and performing pre-processing; (2) computing embeddings and conducting deduplication; (3) categorizing samples and applying basic filtering; (4) annotating data; and (5) grouping samples into training-ready shards based on their resolution and duration."*

### Stage 1 — Raw collection & pre-processing
- Billions of raw images/videos with metadata (raw captions, descriptions) preserved.
- **Scene-change detection via TransNetV2** segments long videos into temporally consistent clips.
- **`ffmpeg cropdetect`** detects and removes **black borders**.
- Re-encode to a canonical format *"to standardize storage and ensure playback integrity."*

### Stage 2 — Embedding & deduplication
Motivated by two problems at once: *"Raw data contains a large amount of repeated image and video content, and its concept distribution is often highly imbalanced."*

| Component | Choice |
|---|---|
| Image embedder | **Qwen3-VL-Embedding-8B** |
| Video embedder | **nvidia/Cosmos-Embed1-448p** |
| Clustering | **cuML KMeans, 20,000 clusters**, fit on a sample of **147M images / 400M video clips** |
| Dedup rule | Assign each sample to its nearest cluster; **near-duplicate removal within each cluster by cosine similarity** |

Clustering-then-within-cluster-similarity is the standard trick for making O(N²) dedup tractable at billion scale — and it doubles as the substrate for **concept balancing**.

### Stage 3 — Categorization & basic filtering
A suite of in-house VLMs performs semantic tagging and quality filtering into **47 hierarchical categories** spanning General and Physical AI domains.

**Image filters:**
- Aesthetic score and photorealism score produced by a dedicated model; **retained only if aesthetic score exceeds a threshold**.
- **Hard-discard tags: collage, watermark, white background, NSFW.**
- Synthetic images not intended for text rendering are additionally filtered by **photorealism score**.

**Video filters — three continuous scores on a 0–9 scale:**
| Score | Source |
|---|---|
| **DOVER aesthetic quality** | Wu et al. 2023 |
| **DOVER technical quality** | Wu et al. 2023 |
| **VTSS training suitability** | Wang et al. 2025 |

plus **~100 binary artifact tags**, with a **two-tier severity policy**:
- **Major artifacts → rejection**: split-screen layouts, rotated videos, static videos.
- **Minor artifacts → flagged but retained** in pre-training: text overlays, motion blur, compression noise.

The severity split is an important, transferable idea: minor degradations are *labelled and kept* (so the model sees realistic imperfection and the flags remain available for later re-filtering), while structurally invalid samples are removed outright.

## 3. Tiered training corpora — pre-train / mid-train / post-train

Quality is not a single gate but a **ladder**, with progressively stricter admission.

### Mid-training image mixture
- Stricter per-aspect-ratio resolution thresholds + a **strict DOVER aesthetic cutoff**
- **Oversampling of hard-case concepts to mitigate the long-tailed distribution**
- Synthetic images *"curated through careful rejection sampling"* broaden coverage of uncommon visual concepts and object compositions
- Text-rendering images added because legible in-image text is underrepresented

**Effective proportions: 60% real / 36% synthetic / 4% text-rendering.**

### Mid-training video mixture
| Component | Share | Purpose |
|---|---:|---|
| Stricter-filtered clips re-selected from the pre-training pool | **46.0%** | Quality lift |
| Domain-specific: robotics, autonomous driving, human activity, egocentric human-object interaction | **43.9%** | Embodied-AI / manipulation targeting |
| **Capability-oriented hard cases**: human motion, high-speed complex motion, fine-grained manipulation | **10.1%** | Corner-case robustness |

Deliberately allocating ~10% of the mixture to *known failure modes* is targeted curriculum design, not generic filtering.

## 4. The VLM-supervision pipeline — semantic dedup + AI judge

For the instruction-tuning ("conversation") data, a separate **two-stage** pipeline:

### Stage A — Semantic deduplication at conversation level
A *conversation* = the complete training example (image/video + instruction-response text, or text-only).
- **Joint embedding** concatenating media and instruction-response text representations — *"enabling the pipeline to distinguish visually similar samples with different task intents from truly redundant supervision."* (Embedding the media alone would over-delete.)
- Image-text and text-only → **Qwen3-VL-Embedding-8B**; video-text → **Perception Encoder PE-Core-G14-448**.
- K-means partition, then within-cluster cosine similarity; **similarity > 0.95 → removed**.
- **Measured effect: removes 4.23% of the data as near-duplicate supervision.**

### Stage B — AI-judge quality filtering
- Judge: **Gemma-4-31B-it**, *"prompted as a training-data auditor"*.
- Rubric-based **integer scores 1–5 on three dimensions**:

| Dimension | Definition | Why it matters here |
|---|---|---|
| **Faithfulness** | Are all response claims grounded in the provided image/video/text? | *"unsupported visual claims can teach the model to hallucinate physical states, object attributes, or temporal events"* |
| **Completeness** | Does the response fully address the instruction without important omissions? | Removes under-specified/partial supervision |
| **Correctness** | Is it factually, logically, and task-level accurate? | Removes inconsistent reasoning/answers |

- The judge also emits **short evidence-based rationales**, *"enabling targeted spot checks and auditing of the filtering behavior"* — the filter is itself auditable.

### Thresholding policy — minimum, not mean
> *"A sample is retained only if its Completeness, Correctness, and Faithfulness scores **all** meet or exceed a specified threshold… This filtering strategy is intentionally stricter than averaging the scores. Samples with a severe failure mode in any single dimension are removed even if they score highly on the remaining criteria."*

Example given: a detailed, logically correct response with unsupported visual claims is still cut on Faithfulness.

### Retention analysis — checking the filter doesn't collapse the skill distribution
> *"we analyze retention by capability category to ensure that quality filtering improves the corpus without unintentionally collapsing the skill distribution."*

Finding: **pruning is strongly category-selective at stricter thresholds.** Referring-expression grounding is removed most aggressively; image captioning and VQA decline substantially as the threshold rises 2 → 5; OCR data is comparatively robust at threshold 2. Hence *"we use conservative thresholding to eliminate clearly low-quality supervision while minimizing excessive distribution shift across capability domains."*

**This is the most important methodological point in the whole survey about filtering: a quality filter is also a distribution shift, and it must be measured as one.**

## 5. SILA — the curation platform

*Scalable Infrastructure for Large-scale data processing and Annotation.* Requirements: (1) distributed raw→training-ready transformation, (2) embedding-based retrieval/clustering/dedup, (3) interactive visualization, inspection, debugging.

**Unified columnar data layer.** Curation is a single **Lance dataset** — one row per sample, one typed column per **curation signal** (caption, tag, quality score, annotation). This replaces the earlier Cosmos-Predict 1.0/2.5 **table-per-pipeline** design (each pipeline writing its own Postgres table, synced to Databricks via CDC), which *"required increasingly complex joins across large tables to reconstruct the state of a single sample."* A concrete, hard-won architectural lesson.

**Vector search co-located with metadata.** Embeddings live in Lance data files, not inline relational rows, so *"large embedding payloads do not inflate metadata tables or slow operational queries."* Production configuration:
- **4096-dimensional embedding column over tens of billions of rows**
- **LanceDB IVF_PQ indexes with cosine similarity**
- **64K IVF partitions** + PQ-compressed embeddings

Because indexes are built over the primary dataset, *"metadata-aware filtering, nearest-neighbor retrieval, and downstream dataset analysis remain synchronized with continuously evolving embeddings, annotations, and curation outputs without requiring synchronization between separate vector and metadata systems."*

**Execution model** — designed for reality rather than an idealized allocation: *"Curation workloads must run continuously on shared clusters with fragmented and dynamically changing GPU availability rather than assuming a single monolithic allocation."* Hence staged Ray execution, fragment-level coordination and fault recovery, node-local model serving, opportunistic cluster utilization, incremental recomputation, and agent-friendly operational interfaces.

Output format: **WebDataset-format training shards**, grouped by resolution and duration.

**The governing philosophy:** *"SILA turns data curation from a sequence of one-off preprocessing jobs into a continuously evolving [substrate]"* — pipelines are rerun *"incrementally as models, quality criteria, and training recipes change."*

## 6. Speech/audio curation
One-line principle, included because it is unusually crisp: *"The curation pipeline is organized around a simple principle: keep speech only when it is synchronized with a [visible source]."* Cross-modal consistency as the sole admission criterion.

## 7. Related NVIDIA numbers (platform-level, from the Cosmos WFM line)
- **20M hours of video → 100M clips** extracted by the curation pipeline.
- NeMo Curator on Blackwell processes **20M hours of video in ~14 days**, vs >3 years CPU-only.
- **Physical AI Data Factory Blueprint** (GTC 2026) connects Cosmos Curator (curation), Cosmos-Transfer (domain adaptation), and Cosmos-Reason/Evaluator (quality assessment) with NVIDIA OSMO as orchestrator.

## 8. What they do not do / limits
- No per-source hour accounting or provenance/licensing table (contrast Hydra-0).
- Aesthetic/photorealism/DOVER/VTSS thresholds are described as "predefined" or "strict" but the actual numeric cutoffs are not published (the 0.95 dedup similarity and the 1–5 judge scale are the exceptions).
- The AI judge is a single model (Gemma-4-31B-it) with no cross-model adjudication (contrast Qwen-RobotManip's multi-VLM voting).
- Filtering is optimized for *generation* quality; the link from aesthetic score to *policy* performance is not established.

## 9. Transferable takeaways
1. **Measure retention by capability category.** A quality filter reshapes your skill distribution; if you don't measure it per category you will silently delete a capability (here: referring-expression grounding).
2. **Minimum-across-dimensions, never the mean.** One severe failure mode should disqualify a sample regardless of its other scores.
3. **Two-tier artifact severity** — reject structural defects, flag-and-keep cosmetic ones.
4. **Embed the *joint* (media + instruction) representation for dedup**, so identical images serving different task intents survive.
5. **Cluster first, then dedup within cluster** — the only tractable route at billion scale (20K clusters here).
6. **One columnar table, one column per curation signal.** The table-per-pipeline architecture does not survive contact with a dozen pipelines.
7. **Design for low yield.** If 90% of candidates die, order your stages so the cheap ones kill samples before the expensive ones touch them.
8. **Make the filter auditable** — judge rationales enable spot checks that a bare score cannot.
9. **Budget explicitly for hard cases** (10.1% of the mid-training video mixture here).
