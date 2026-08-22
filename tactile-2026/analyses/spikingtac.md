# SpikingTac — A Miniaturized Neuromorphic Visuotactile Sensor for High-Precision Dynamic Tactile Imprint Tracking

**arXiv:2602.23654** · Institute of Automation CAS + UCAS (T. Jiang, C. Zhang, S. Zhang, Cui, S. Wang) · Feb 2026

**One line.** Builds a **custom standalone event camera module for under $150** to make neuromorphic tactile small enough to mount, then solves the problem event sensors expose: **silicone viscoelastic hysteresis**.

## 1. The obstacles

Event-driven tactile sensors are *"essential for achieving human-like dynamic manipulation, yet their integration is often limited by the **bulkiness of standard event cameras**."* Hence the custom module, at **< $150 total material cost**.

The second obstacle is physical rather than electronic: **viscoelastic hysteresis of silicone elastomers**. An event sensor measures *change*, so a gel that does not return to its rest state accumulates drift — the sensor's zero point wanders.

## 2. Method

- **Global dynamic state map** with an **unsupervised denoising network**, giving **1000 Hz perception** and **350 Hz tracking**
- **Hysteresis-aware incremental update law** with a **spatial gain damping mechanism**, directly addressing elastomer viscoelasticity

## 3. Results

| Metric | Value |
|---|---|
| Return-to-origin success | **100%**, mean bias **0.8039 px**, even under extreme torsional deformation |
| Obstacle-avoidance overshoot | **6.2 mm** — **5× better** than conventional frame-based sensors |
| Localisation RMSE | **0.0952 mm** |
| Radius measurement RMSE | **0.0452 mm** |

The zero-point stability result is the one that matters for deployability: an event tactile sensor that drifts is unusable over a long episode, and 100% return-to-origin under torsion means the hysteresis compensation works where it is hardest.

The 5× overshoot reduction on obstacle avoidance is the direct demonstration of what temporal bandwidth buys — the same argument [[mitas]] makes by fusing an Evetac alongside a GelSight, here delivered by a single sensor.

## 4. What it adds that the others don't

**Making event-based tactile practical.** The modality has clear advantages for the fast-contact regime that [[mitas]] and [[vibeact]] both target, and its adoption has been blocked by camera size and by drift. A **$150 mountable module with 0.8-pixel zero-point stability** removes both. Sub-0.1 mm geometric accuracy at 1000 Hz also makes it one of the few sensors here that could support the high-frequency reactive loops that [[tacmamba]], [[t-rex]] and [[omnivta]] design around.
