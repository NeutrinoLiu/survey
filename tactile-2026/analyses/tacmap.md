# Tacmap — Bridging the Tactile Sim-to-Real Gap via Geometry-Consistent Penetration Depth Map

**arXiv:2602.21625** (v2, May 2026) · **Sharpa** + HKUST + NVIDIA (L. Su, Z. Peng, Ren, Mao, J. Du, K. Zhang, X. Zhu) · Feb 2026

**One line.** Sidesteps the sim-to-real gap by refusing to compare *images*: both simulation and reality are mapped into a shared **deformation-map** space, because *"while raw tactile images are sensor-specific and optically complex, the underlying deformation map serves as a universal proxy for contact physics."*

## 1. The trilemma in tactile simulation

Three existing families, each failing differently:

| Family | Example | Failure |
|---|---|---|
| **Empirical** | Taxim | *"dependency on specific data distributions often leads to poor generalization toward novel object geometries"* |
| **Analytical** | TACTO | simple depth-buffer rendering *"fails to capture the physical deformation and volume-preserving characteristics of the elastomer"* |
| **Physics-based** | TacSL, TacEx | *"extreme computational costs and the difficulty of matching complex material parameters"* |

Plus a form-factor problem the field mostly ignores: *"most existing simulation toolkits are implicitly designed for flat-surface sensors. When applied to **curved fingertips**, which are increasingly common in anthropomorphic hands, these methods struggle with projection distortions and the non-planar nature of elastomer deformation."*

## 2. Method — meet in the middle

**In simulation:** an efficient geometric computation of the **3D penetration depth / intersection volume** between rigid object and sensor elastomer, synthesised into a deform map — *"capturing the essential deformation features without the overhead of FEM."*

**In reality:** an **automated data-collection rig** measuring ground-truth deformation during physical interaction, used to train a translation model from **raw tactile images → depth maps**.

**Both domains therefore live in the same geometric space**, minimising domain shift while preserving physical consistency.

**Geometry-agnostic by design** — calculations are performed in a *generalised normal-projection space*, supporting flat *and* curved sensor surfaces.

## 3. Results

Quantitative comparison across diverse contact scenarios shows Tacmap's deform maps *"closely mirror real-world measurements."* Validated end to end on an **in-hand ball rotation** task where a policy **trained exclusively in simulation transfers zero-shot to a physical robot**. Real-world interaction data can also be **replayed inside the simulator** for verification — a useful debugging affordance.

## 4. What it adds that the others don't

**Choosing the right representation to align, rather than trying to close the gap in pixel space.** This is the same move as [[uniforce]] (align in force space) and [[fg-cltp]] (align in 3D point-cloud space), applied to simulation: pick the quantity that is invariant across the domains you need to bridge, and render into it from both sides. Penetration depth is a good choice because it is what the elastomer physically encodes and what neither the optics nor the sensor geometry perturbs.

The explicit support for **curved fingertips** also matters more than it sounds: anthropomorphic hands are curved, and a simulator that assumes flat gels excludes exactly the hardware the dexterous-manipulation field is moving toward.
