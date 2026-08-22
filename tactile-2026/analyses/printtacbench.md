# A 3D-Printable Dataset for Fair Testing and Comparisons of Tactile Sensors

**arXiv:2606.25886** · University of Sussex (Shepherd, Herzig, Husbands, Philippides, Johnson, Kimbell) · Jun 2026

**One line.** Distributes the *stimulus* rather than the sensor readings — six mathematically defined textures anyone can print — and then measures how badly 3D printing itself corrupts the benchmark.

## 1. What "tactile" means here

Optical tactile imaging via a **TacTip** (marker-displacement) sensor under controlled contact. But tactile is secondary; the object of study is the **texture standard**.

The critique that motivates it is sharp and correct: existing texture datasets are *sensor readings from one sensor touching whatever objects were lying around*. They can be used for model optimisation or transfer learning but never for comparing a new sensor against a published one, because the stimulus cannot be reproduced.

## 2. Data curriculum

**Six parametric surfaces**, each a sum of sine waves so that spatial frequency, amplitude and directional structure are exactly controllable — modelled on the USAF resolution test chart rather than on scanned real surfaces (3D scans were tried first and rejected: non-uniformity *within one sample* meant sensor response depended on contact location).

- **T1/T2** — `z = A₁sin(f₁x) + A₂sin(f₂y + φ)`; T1 with A₂ = 0 (1-D grating), T2 with A₂ = 1 (2-D).
- **T3** — odd-harmonic Fourier series, `z = Σ Aᵢ(sin(ix) + sin(iy))`, N = 25, `Aᵢ = (8/π²)·(−1)^((i−1)/2)/i²` — a symmetric square-wave-like pattern.
- **T4** — same series, coefficient `Aᵢ = 1/(πi)`: no sign alternation, sharper ridges and edges.
- **T5/T6** — extruded (y-invariant) versions of T3 and T4.

Units are left unspecified so the set can be printed at any scale (only one size validated). Everything exports to STL.

The dataset proper is the **variance study**: TacTip images collected across three printers and multiple filament types, under controlled contact.

## 3. Model

Random forest and a small ANN, plus PCA-based models. The classifiers are instruments for measuring print reproducibility, not contributions.

## 4. How tactile enters the model

Directly as TacTip images. No fusion.

## 5. Experiment setup

Train a texture classifier on one printer–filament batch, test (a) held-out images from the same batch (20% split), and (b) **all other printer–filament combinations**. Averaged over 20 trials.

## 6. Results — and the uncomfortable finding

| Model | Train | Test (same batch) | **Unseen printer/filament** |
|---|---|---|---|
| RFC, Ender-3 V3 SE | 100.0% | 99.53% | **31.4%** |
| ANN, Ender-3 V3 SE | 99.7% | 99.3% | **33.2%** |
| RFC, Bambu only | 100% | 99.7% | 63.98% |
| ANN, Bambu only | 100% | 99.8% | 53.73% |
| RFC, Resin | 100% | 99.3% | **70.66%** |
| ANN, Resin | 100% | 99.9% | 61.41% |

Within-batch accuracy is ~99.5% everywhere; cross-printer accuracy collapses to **31–71%** against a 16.7% chance floor. The classifiers are partly learning **print defects, not texture** — the authors say so directly: *"Classifiers start to recognise the slight differences of various prints which aid in the classification task."*

Confusion concentrates on genuinely similar pairs (T2/T3, T1/T5), and what distinguishes them in practice is sometimes a peak that failed to print sharply.

**Hardware recommendation, empirically derived:** resin gives lowest variance and best cross-print generalisation; Bambu Lab + PLA+ is the strong lower-cost filament option; low-cost systems (Ender-3 V3) show stringing and blunted peaks that dominate the signal. Print quality — specifically peak sharpness and stringing — is the variable that determines tactile variance.

**Limits.** Only one texture size validated; three printers; no slicer-level optimisation or printer calibration explored; TacTip only, so the cross-*sensor* comparison the dataset is built to enable is not itself performed.

## 7. What it adds that the others don't

The only work in this survey that treats **the physical stimulus as the thing needing standardisation**, and the only one to quantify manufacturing variance as a confound in tactile benchmarking. Its negative result — a 99.5% classifier dropping to 31% across printers — is a caution that applies to every 3D-printed indenter set in the literature, including the shape and grating subsets of [[tacverse]].
