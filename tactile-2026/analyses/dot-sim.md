# DOT-Sim — Differentiable Optical Tactile Simulation with Precise Real-to-Sim Physical Calibration

**arXiv:2604.27367** · Stanford University + University of Cambridge (Y. You, W. K. Do, Swann, Antonova, Kennedy, **Guibas**) · Apr 2026

**One line.** Models the gel as **elastic material via the Material Point Method**, calibrates it from a handful of real demonstrations **in minutes**, and handles optics by learning a **residual image relative to the real sensor's idle state**.

## 1. The two halves of the problem

Optical tactile simulation must get both **deformation physics** and **internal light transport** right. Existing frameworks *"often rely on simplified approximations of contact geometry or assume fixed mappings from force to image output"*, which fails because *"many optical tactile sensors with flexible surface materials exhibit highly nonlinear responses to contact, due to both material elasticity and internal optical effects."*

## 2. Method

**Physics** — the sensor is modelled as an elastic material with **MPM**, supporting *"much larger and non-linear deformations"* than baselines. Being **differentiable** enables *"rapid calibration... using a small number of demonstrations within minutes, which is substantially faster than existing methods."*

**Optics** — rather than simulating light transport, learn a **residual image relative to the real-world idle state**. The sensor's own no-contact appearance carries all the optical idiosyncrasy; only the *change* needs to be modelled. (The same non-contact/contact decomposition [[unitac]] uses for generation and [[n0-vtla]] uses for encoding.)

## 3. Evaluation — a well-designed metric suite

**Physical fidelity**, on the deformed surface point cloud:
- **Chamfer Distance** and **Earth Mover's Distance** — average-case, EMD *"more sensitive to global shape structure"*
- **Significant CD** — top 1% largest nearest-neighbour distances, *"emphasizing outliers and surface discrepancies"*
- **F-Score @ 1 mm**

**Optical fidelity**:
- Mean pixel L2, **Significant Pixel L2** (top 1% worst-predicted pixels), and PSNR

Reporting both average-case and **worst-case** metrics is unusual and appropriate — a tactile simulator that is right on average and wrong at the contact boundary is wrong where it matters.

## 4. Zero-shot sim-to-real results

Validated on a **DenseTact** sensor across four claims:

| Task | Result |
|---|---|
| Object classification, sim-trained classifier deployed real | **85%** on challenging objects |
| **Embedded tumour-type detection** | **90%** |
| Trajectory following, policy from sim demonstrations | **< 0.9 mm** average error |
| Physical dynamics replication | accurate |

The tumour-detection result is worth noting — palpation for embedded stiffness inhomogeneity is a medical application, and one of the few non-manipulation uses of tactile in this survey alongside [[vitatouch]]'s industrial inspection.

## 5. What it adds that the others don't

**Differentiability, and therefore fast calibration.** Minutes-from-a-few-demonstrations is a different operating model from the hand-tuned material parameters that [[tacmap]] identifies as a barrier for physics-based simulators. The **residual-to-idle** optical trick sidesteps light-transport modelling entirely, and the worst-case metrics (Sig. CD, Sig. Pixel L2) are a reporting practice the rest of the simulation cluster should adopt.
