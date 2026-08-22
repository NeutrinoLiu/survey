# Cortex 2.0 — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Cortex 2.0: Grounding World Models in Real-World Industrial Deployment |
| **Org** | Sereact GmbH |
| **Date** | 2026-04-22 |
| **Artifact** | arXiv 2604.20246 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **B — full paper, but data-side detail is thin; the contribution is planning, not curation** |
| **Stance** | **Generate candidate futures at inference, score them, commit to the best.** Data selection moved from training time to test time. |

## 1. Why it appears in this survey

Cortex 2.0 is included as the **industrial-deployment counterpoint**: it is the entry most directly concerned with what happens when a policy trained on curated data meets an uncurated environment.

Its diagnosis of VLAs is a claim about the limits of training-data curation:
> *"While Vision-Language-Action models have demonstrated strong generalization, they remain **fundamentally reactive**. By optimizing the next action given the current observation **without evaluating potential futures**, they are **brittle to the compounding failure modes of long-horizon tasks**."*

The remedy relocates the selection problem:
> *"Cortex 2.0 shifts from reactive control to **plan-and-act** by **generating candidate future trajectories in visual latent space, scoring them for expected success and efficiency, then committing only to the highest-scoring candidate**."*

This is **rejection sampling at inference time** — structurally the same operation as RoboCurate's Best-of-N generation filtering and Tencent's rejection-sampling SFT, but applied to actions about to be executed rather than to training samples. It is the natural endpoint of the survey's recurring theme: *generate many candidates, score them, keep the good ones.* Cortex applies it where the stakes are highest.

## 2. Evaluation conditions — a description of uncurated reality

Single-arm and dual-arm platforms, four tasks of increasing complexity: **pick and place items · item and trash sorting · screw sorting · shoebox unpacking.**

> *"The system remains reliable in **unstructured environments characterized by heavy clutter, frequent occlusions, and contact-rich manipulation, where reactive policies fail**."*

This matters for data practice because it names the deployment distribution that curated benchmark data typically excludes. Heavy clutter and frequent occlusion are exactly the conditions under which the perception-derived labels that most 2026 pipelines depend on (SAM3 masks, HaMeR hand poses, SLAM trajectories, depth estimates) degrade — and Hydra-0 independently reports that reliable object masks *"cannot be obtained in [DROID's] cluttered scenes."*

The industrial task set is also notable: **screw sorting** and **shoebox unpacking** are the kind of long-tail, low-glamour tasks that a corpus assembled from academic benchmarks or household video will not contain, which is a coverage argument for domain-specific collection.

## 3. Reported result
> *"Cortex 2.0 consistently outperforms state-of-the-art Vision-Language-Action baselines, achieving the best results across all tasks… These results demonstrate that our **world-model-based planning can operate reliably in complex industrial environments**."*

## 4. What they do not do
- No pretraining corpus description: no sources, hours, mixture, filtering, or annotation pipeline.
- No data-scaling evidence.
- The trajectory scorer (expected success and efficiency) is the analogue of RECAP's value function and RoboBrain's progress estimator, but its training data is not described.

## 5. Transferable takeaways
1. **Test-time selection substitutes for some training-time curation.** If you can score candidate futures, you need not have curated a corpus that makes the first candidate correct.
2. **Score on efficiency as well as success.** Most quality signals in this survey are binary or success-only; industrial deployment cares about throughput, and Cortex scores both.
3. **Deployment conditions define the coverage gap.** Heavy clutter and occlusion break the perception-derived labelling that most 2026 data pipelines rely on — a reason to expect curated-corpus quality to degrade exactly where deployment happens.
4. **Industrial task distributions are absent from public corpora.** Screw sorting and shoebox unpacking do not appear in household-video or academic-benchmark data at any scale.
