# RoboCasa365 — Simulation Data Generation & Benchmark Design

## 0. Card

| Field | Value |
|---|---|
| **Work** | RoboCasa365: A Large-Scale Simulation Framework for Training and Benchmarking Generalist Robots |
| **Date** | 2026-03 |
| **Artifact** | arXiv 2603.04356 (`paper.pdf`, `paper.html`) |
| **Disclosure level** | **A — full paper; framework and datasets released** |
| **Corpus** | **300+ tasks · 2,500 unique kitchen environments · 500K+ demonstrations · 600+ h human demonstration · 1,600+ h synthetically generated** |
| **Stance** | **Simulation as the instrument for studying what data properties matter.** |

## 1. Why it belongs in a data-curation survey

RoboCasa365's contribution is not a cleaning pipeline but the **experimental apparatus for answering data questions at all**:

> *"it remains difficult to study **how task diversity, environment variation, and dataset scale affect policy generalization**."*

Real-world evaluation is *"resource-intensive and time-consuming… often affected by experimental noise, making it difficult to perform reproducible, systematic comparisons."* Simulation *"enables rapid experimentation, controlled evaluation, and reproducible benchmarking that would be infeasible in real-world robotics."*

Every data-mixture claim in this survey is bottlenecked on evaluation cost. RoboCasa365 is infrastructure for lifting that bottleneck.

## 2. Scale relative to prior simulation corpora

| Framework | Demonstrations | Tasks | Scenes |
|---|---:|---:|---:|
| RoboCasa (Nasiriany et al. 2024) | 100K | 30 | 100 |
| **RoboCasa365** | **500K+** | **300+** | **2,500** |

> *"Our work is unique in that it features **hundreds of tasks across thousands of unique scenes, large-scale, high-quality demonstration datasets, and a suite of benchmarks** for training and evaluating generalist robot models. To our best knowledge, our work is the first simulation framework to satisfy all of these criteria."*

Composition: **600+ hours human demonstration + 1,600+ hours synthetic** — a ~1:2.7 real-to-synthetic ratio within simulation, comparable to GR00T N1's ~1:8.4 neural augmentation multiplier and Qwen-RobotManip's ~1:12.8 H2R multiplier.

## 3. Designed for three distinct data-study settings

> *"RoboCasa365 is designed to support **systematic evaluations for different problem settings, including multi-task learning, robot foundation model training, and lifelong learning**."*

> *"we aim to be **agnostic to the choice of model**, and instead create benchmarks to systematically assess the capabilities of these models across distinct settings, including multi-task training, **pretraining, and fine-tuning on target data**, and lifelong learning."*

Model-agnostic benchmark design is what makes cross-work comparison possible — the exact gap the [VLA data survey](../vla-data-survey/dataprocess.md) identifies (*"different works use different task definitions, success criteria, and data splits"*).

## 4. The data findings

> *"We conduct extensive experiments on this benchmark with state-of-the-art methods and **analyze the impacts of task diversity, dataset scale, and environment variation on generalization**. Our results provide new insights into **what factors most strongly affect the performance of generalist robots**."*

This decomposition — **task diversity vs. dataset scale vs. environment variation** as three separable axes — is the same factorization EgoVerse builds its 16×16 demonstrator×scene assignment matrix to study, and that EgoVerse independently resolves in favour of environment/scene diversity past a volume threshold.

## 5. Use as a baseline elsewhere in this survey
- **AXIS** benchmarks its own corpus against a **volume-matched RoboCasa365 continual-pretraining baseline**, and reports outperforming it by 37.3% — the kind of composition-isolating comparison that only a released, well-specified corpus makes possible.
- **Qwen-RobotManip** adopts RoboCasa365 as one of its OOD evaluation settings, precisely because standard in-domain benchmarks *"systematically fail to capture the quality of pretraining."*

Both uses illustrate the point: a good simulation corpus functions as a **measuring instrument** for other people's data decisions.

## 6. Limitations
- Kitchen domain only; environment variation is broad but semantically narrow.
- Simulation fidelity bounds what conclusions transfer to real robots — the *"shared limitations in physical grounding and sim-to-real transfer"* the VLA data survey identifies for all data engines.
- Synthetic demonstrations inherit whatever biases the generation procedure has; no verification stage analogous to RoboCurate's simulator replay is reported.

## 7. Transferable takeaways
1. **Build the measuring instrument before making data claims.** Real-robot evaluation is too noisy and expensive to support mixture ablations; simulation is how data science becomes affordable in robotics.
2. **Separate task diversity, environment variation, and dataset scale** as independent experimental axes — they are routinely conflated under "dataset size".
3. **Make benchmarks model-agnostic** so that data claims from different groups become comparable.
4. **Publish volume-matched baselines** so others can isolate composition from quantity (as AXIS does against RoboCasa365).
