# Tac2Real — Reliable and GPU Visuotactile Simulation for Online RL and Zero-Shot Real-World Deployment

**arXiv:2603.28475** · Shanghai AI Laboratory + HKUST + SJTU (N. Yan, S. Wang, X. Shen, H. Wang, H. Wang, Y. Xiang, Pang) · Mar 2026 · [site](https://ningyurichard.github.io/tac2real-project-page/)

**One line.** Targets **online RL** specifically — where the simulator must produce tactile at interactive rates inside the training loop — and pairs a fast IPC solver with **TacAlign**, a systematic treatment of both *structured* and *stochastic* domain gap.

## 1. The requirement

*"Integrating VBTS into data-driven policy training represents a key challenge for online RL. The fundamental limitation lies in efficiently synthesizing high-fidelity simulated tactile signals."*

The two existing families fail in opposite directions: **penalty-based** approaches scale but *"fall short of modeling soft deformation and multi-phase contact dynamics that underpin realistic VBTS sensing"*; high-fidelity physics is too slow for the RL loop.

## 2. Method

**Simulation** — **PNCG-IPC** (Preconditioned Nonlinear Conjugate Gradient Incremental Potential Contact) on a **multi-node, multi-GPU high-throughput parallel architecture**, generating **marker displacement fields at interactive rates**, with **cross-engine compatibility** so it plugs into existing physics engines.

**TacAlign** — a systematic sim-to-real reduction along two axes:
- **Structured gap** — **parameter alignment** (Young's modulus E, Poisson's ratio ν, density ρ, friction μ) and **control alignment**
- **Stochastic gap** — **randomisation**

Separating the two is the useful part: parameter mismatch is a bias to be calibrated away, sensor and contact variability is a variance to be randomised over, and treating both with the same tool works poorly.

## 3. Results

Evaluated on **peg insertion**, with **zero-shot transfer achieving a high real-world success rate**. The ablation confirms *"parameter-based calibration plays an important role in narrowing the sim-to-real gap, without which the success rate will decrease significantly."*

## 4. Stated future directions

Dexterous-hand and deformable-object RL; **large-scale high-quality data for tactile VLA training**; AI-accelerated tactile simulation. And a broader claim about the architecture: *"the architecture of Tac2Real can be generalized to any physical sensor simulation, including tactile, acoustic or even olfactory sensor simulation."*

## 5. What it adds that the others don't

**Marker displacement fields at online-RL rates**, and the **structured/stochastic decomposition** of the domain gap. Where [[tacmap]] aligns sim and real by choosing a shared representation, [[dot-sim]] by differentiable calibration, and [[tactspace]] by a learned shared latent, Tac2Real does it by explicitly identifying which physical parameters to fit and which residual variation to randomise — the most conventional approach, and the one that generalises most directly to new sensors.
