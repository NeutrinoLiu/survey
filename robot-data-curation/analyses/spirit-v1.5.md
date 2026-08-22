# Spirit-v1.5 (Spirit AI) — Pretraining Data Curation & Cleaning Pipeline

## 0. Card

| Field | Value |
|---|---|
| **Work** | Spirit-v1.5: A Robotic Foundation Model |
| **Org** | Spirit AI (千寻智能) |
| **Date** | 2026-01 (initial release); 2026-04 fine-tuning code |
| **Artifacts** | `README.md` (github.com/Spirit-AI-Team/spirit-v1.5); HF `Spirit-AI-robotics/Spirit-v1.5`; technical blog |
| **Disclosure level** | **B — open code, base + fine-tuned checkpoints, technical blog; no data corpus disclosure** |
| **Architecture** | **Qwen3-VL backbone + DiT head + policy API** |
| **Benchmark** | **#1 on RoboChallenge Table30 as of 2026-01-11** |

## 1. Why it is included

Spirit-v1.5 is here mainly as a **benchmark reference point** and as an example of the field's emerging release norm. Its headline claim is competitive standing on a shared external benchmark:

> *"As of Jan 11, 2026, Spirit-v1.5 ranks **#1** on the RoboChallenge Table30 benchmark."*

**RoboChallenge Table30** matters for this survey because it is one of the few *shared, externally administered* evaluations in 2026 robot manipulation — and shared evaluation is the precondition for comparing data strategies at all. Qwen-RobotManip later reports ranking **1st on the RoboChallenge Table30-v1 generalist track with a 20% relative improvement**, i.e. the two claims are directly comparable in a way that most results in this survey are not.

This is precisely the gap the [VLA data survey](../vla-data-survey/dataprocess.md) identifies: *"different works use different task definitions, success criteria, and data splits, which makes it difficult to compare methods."* RoboChallenge is part of the answer.

## 2. What the release tells us about data handling

The repository structure discloses the data path even though the corpus is not described:

```
spirit-v1.5/
├── model/
│   └── modeling_spirit_vla.py    # Qwen3-VL backbone + DiT head + policy API
├── dataset/
│   ├── dataset.py                # Dataset implementation
│   └── transforms.py             # Data transformations
├── utils/                        # checkpointing, distributed, logging
```

Releasing `dataset.py` and `transforms.py` alongside a **base checkpoint, a fine-tuned checkpoint, and (from April 2026) fine-tuning code** means the *preprocessing and augmentation* are reproducible even when the corpus is not — the same partial-transparency posture as Wall-OSS (LeRobot data-prep path released, corpus not) and GR00T (data released too, the stronger position).

## 3. Architectural note relevant to data
**Qwen3-VL backbone + DiT action head** is the dominant 2026 architecture, shared with Qwen-VLA (Qwen3.5-4B + DiT), ABot-M0, Hy-Embodied, and ACE-Ego-0. Convergence on a common backbone family means differences in reported performance increasingly reflect **data and training recipe rather than architecture** — which is exactly why the curation question has become the field's live one.

## 4. What is not disclosed
- No corpus size, sources, mixture, or hours.
- No filtering, annotation, or quality-control pipeline.
- No data-scaling evidence.
- The technical blog is the only narrative artifact; there is no paper.

## 5. Transferable takeaways
1. **Report on a shared external benchmark.** RoboChallenge Table30 is one of the few places where 2026 data strategies can be compared on equal terms.
2. **Release `dataset.py` and `transforms.py` even if you cannot release the corpus** — preprocessing and augmentation are a meaningful part of reproducibility, and they are cheap to publish.
3. **When architectures converge, performance differences are data differences.** The Qwen3-VL + DiT consensus makes the corpus the remaining independent variable.
