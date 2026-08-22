# GR-RL (ByteDance Seed) — release artifact

**This entry is covered in full within the ByteDance Seed stream analysis:**
### → [GR-3 / GR-RL — Pretraining Data Curation & Cleaning Pipeline](../gr-3/dataprocess.md)

## Card

| Field | Value |
|---|---|
| **Work** | Seed Research: GR-RL — real-world reinforcement learning for high-precision VLA manipulation |
| **Org** | ByteDance Seed (Seed-Robotics) |
| **Date** | 2026 |
| **Artifact** | `page.html` (seed.bytedance.com blog) |
| **Disclosure level** | ⚠️ **C — company blog.** No corpus or pipeline description. |

## The headline result

Applying real-world RL to shoe-lacing, a high-precision contact-rich task:

| Model | Shoe-lacing success |
|---|---|
| **GR-3** (supervised) | 45.7% |
| **GR-RL** | **83.3%** |

*"reducing failures by nearly 70%."*

## Why it matters for data practice

GR-RL moves the ByteDance stream into the same regime as **π*₀.₆/RECAP**, **1X's NEO flywheel**, and **π₀.₇'s autonomous-rollout ingestion**: the training signal comes from **the policy's own on-robot experience** rather than from human demonstrations.

That shift changes the curation question entirely. On-policy data is:
- **Abundant** — it grows with deployment rather than with operator hours
- **Mixed-quality by construction** — it contains the current policy's failures
- **Self-referential** — its distribution shifts as the policy improves

Every work in this survey that ingests it has needed an explicit grading or conditioning mechanism (RECAP's learned advantage, π₀.₇'s quality metadata, AGIBOT's annotated error-recovery trajectories, ACE-Brain-0.5's deviation metadata). ⚠️ ByteDance publishes the *result* of real-world RL but not the mechanism by which the experience is filtered, graded, or weighted.

## Organizational note
ByteDance's Seed foundation-model division restructured in 2026 to create a dedicated **Pretrain Data team** (led by Li Chenggang), formed by merging pretraining teams previously spread across text, code, vision, and speech, now responsible for multi-modal data and large-scale pretraining for the Omni model. Robotics data sits alongside that reorganization — an institutional signal that pretraining data is being treated as a first-class function rather than a per-project concern, which is exactly the shift the [VLA data survey](../vla-data-survey/dataprocess.md) argues the field needs.

See [GR-3 / GR-RL](../gr-3/dataprocess.md) for the full pipeline analysis, including the collection scheduler, viewpoint-shortcut mitigation, compliance-control-as-filter, and the Invalid-Tasks evaluation.
