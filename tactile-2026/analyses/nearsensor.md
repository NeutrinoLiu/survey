# Near-sensor Computing for Rapid Visuotactile Perception

**arXiv:2608.05725** · ShanghaiTech University (Z. Zhu, R. Zhang, R. Hu, **C. Xiao**) · Aug 2026

**One line.** Moves the **Poisson solve onto FPGA next to the sensor**, cutting a robot's protective reflex loop from **170 ms to 28 ms** — a systems fix for a bottleneck the tactile learning literature treats as fixed.

## 1. The bottleneck

Visuotactile sensors reconstruct dense 3D contact geometry by **photometric stereo plus Poisson integration**, and those depth maps support contact-force distribution, surface curvature, incipient slip and object pose estimation.

But: *"The principal bottleneck is 3D reconstruction: recovering contact geometry typically requires solving a Poisson partial differential equation over dense gradient fields, a computation that is **substantially more expensive than the processing used in most other tactile modalities**. On general-purpose CPUs, reconstruction often takes tens of milliseconds, with further variability arising from **operating-system scheduling and host-based data transfer**."*

Two costs, and the second is the subtler one: host processing gives **non-deterministic** latency, which is worse than slow latency for a reflex loop.

## 2. Method

A **spectral Poisson solver as a fully streaming hardware pipeline**, with **no data-dependent branching and no iterative convergence** — hence **deterministic** latency by construction.

| Metric | Value |
|---|---|
| Core logic power | **347 mW** |
| Clock | 166 MHz |
| First depth value of a 128×128 frame | **35,107 cycles = 0.211 ms** fixed |
| Accuracy vs. double-precision reference (15 geometries) | **0.17%** of peak contact depth |

## 3. The result that matters

On-chip decisions from these reconstructions close a **robot protective reflex loop in 28.3 ± 4.9 ms**, against **169.9 ± 27.8 ms** for an equivalent host-based loop **using the same actuator**. 50 trials near-sensor, 81 host-loop, pooled across sessions, two-sided Mann–Whitney U test.

A **6× reduction**, with the standard deviation shrinking from 27.8 to 4.9 ms — the determinism gain is as large as the latency gain.

## 4. What it adds that the others don't

**A hardware answer to the reactivity problem the policy literature keeps hitting in software.** [[tacmamba]] engineers a state-space model to keep a 100 Hz loop; [[t-rex]] builds a variable-rate MoT; [[omnivta]] runs a 60 Hz reflex controller; [[tubedp]] restructures diffusion to reduce denoising steps. All of them are working around a perception pipeline that takes tens of milliseconds with variable jitter. Near-sensor computing removes that constraint at its source — 0.211 ms fixed, at 347 mW — and reframes what control frequencies are achievable at all.

It is also, with [[spikingtac]], one of two 2026 works arguing that the tactile *bandwidth* ceiling is an engineering artifact rather than a property of the modality.
