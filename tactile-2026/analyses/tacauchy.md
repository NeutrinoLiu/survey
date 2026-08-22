# TaCauchy — An Extensible FEM Framework for Vision-Based Tactile Simulation

**arXiv:2606.20426** · **IROS 2026** · Tsinghua SIGS + Huawei (H. Zhao, Y. Xie, Gong, Y. Sun, K. Zhu, W. He, S. Li, H. Fu, W. Ding) · Jun 2026

**One line.** Computes **Cauchy stress tensors from hyperelastic constitutive laws** inside Isaac Sim, giving force ground truth *from first principles* rather than empirical estimation — and still runs at 555 FPS aggregate.

## 1. The problem

*"How to obtain accurate and reliable force information through rigorous mechanical computation while maintaining the computational efficiency required for large-scale parallel training, and simultaneously provide high-fidelity visual rendering."*

Three failure modes in prior work: simulators **outside the Isaac ecosystem** (limiting large-scale RL), simulators that **simplify contact models or lack force extraction**, and **purely optical simulators** that cannot provide force at all.

## 2. Method

- **UIPC** (Unified Incremental Potential Contact) solver for soft-body dynamics — intersection- and inversion-free large-deformation contact
- **Cauchy stress tensors** computed directly from **hyperelastic constitutive laws** (stable Neo-Hookean), then **projected onto contact surfaces** to obtain traction forces and pressure distributions
- **Automatic mesh generation with geometry-aware adaptive refinement**
- **Modular sensor interface** — GelSight Mini, DIGIT, 9DTact with minimal configuration
- Integrated into **Isaac Sim**

## 3. Results

| Metric | Value |
|---|---|
| Single environment | **33.40 FPS** |
| 60 parallel environments | **555 FPS** aggregate |
| Stress extraction overhead | **< 1 ms** |
| Sim–real agreement, 1.2556–4.7332 N | **SSIM > 0.93** |

The sub-millisecond stress extraction is the notable engineering figure: the physically rigorous part is not the bottleneck.

## 4. What it adds that the others don't

**Mechanical ground truth from constitutive law.** Every other simulator in this cluster produces a *proxy* — penetration depth ([[tacmap]]), learned deformation propagation ([[etac]]), marker displacement ([[tac2real]]) — whereas TaCauchy produces the actual stress field, which is what force supervision ought to be regressed against. That makes it the natural data source for force-grounded representation work ([[uniforce]], [[taf-vla]], [[fg-cltp]]), which currently relies on instrumented rigs to obtain the same quantity.

The trade-off is scale: 60 parallel environments against [[tactilegenesis]]'s 20,000 and [[etac]]'s 4,096. TaCauchy is the fidelity end of the cluster's efficiency/accuracy spectrum.
