# GR00T N1.7 — release artifact

**This entry is covered in full within the GR00T stream analysis:**
### → [GR00T N1 / N1.7 — Pretraining Data Curation & Cleaning Pipeline](../groot-n1/dataprocess.md)

## Card

| Field | Value |
|---|---|
| **Work** | NVIDIA Isaac GR00T N1.7 (`nvidia/GR00T-H-N1.7`) |
| **Org** | NVIDIA |
| **Date** | 2026 |
| **Artifact** | `README.md` (HuggingFace model card) |
| **Disclosure level** | ⚠️ **B — model card / release notes only.** No technical report accompanies N1.5, N1.6, or N1.7; the last full data disclosure in this stream is **GR00T N1** (arXiv 2503.14734). |

## Why it is tracked separately

N1.7 is the **latest** model in the GR00T stream and the version other 2026 works benchmark against — **Qwen-RobotManip** compares against GR00T-N1.7, and **ACE-Ego-0** explicitly *"adopt[s] the N1.7 version to leverage its latest optimizations for physical deployment"* for real-robot comparison. It is therefore the de facto open baseline for the field.

## Data-side status

The corpus description remains the **data pyramid** documented for N1:
- **Base**: web data + human egocentric video (labelled via latent-action codebook and IDM pseudo-actions)
- **Middle**: synthetic — physics simulation (Mimic pipeline in Isaac Lab) + neural-generated trajectories
- **Top**: real-robot teleoperation (88 h → 827 h after neural augmentation)

⚠️ **What changed between N1 and N1.7 on the data side is not publicly documented.** Given that N1.7 is the reference baseline in most 2026 comparisons, this is a notable gap: papers report beating "GR00T-N1.7" without a public account of what data that model saw.

## Takeaway
**A widely used baseline should carry a data disclosure.** The GR00T stream's openness at N1 (checkpoint + training data + benchmarks released) is what made it the field's reference point; the absence of equivalent documentation for N1.5/N1.6/N1.7 weakens the comparability of every result measured against them.

See [GR00T N1 / N1.7](../groot-n1/dataprocess.md) for the full pipeline analysis.
