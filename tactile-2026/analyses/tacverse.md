# TacVerse — A Multi-Sensor Dataset and Benchmark for Cross-Sensor Vision-Based Tactile Perception

**arXiv:2606.25877** · Imperial College London + QMUL + KCL (Wei, Khurana, Bhouri, … Zhang) · Jun 2026 · [code](https://github.com/LannWei/Tactile_Database) · [HF dataset](https://huggingface.co/datasets/Lan-2025/Tactile)

**One line.** Holds architecture fixed and varies only the sensor, and finds that cross-sensor tactile transfer is not merely degraded but *catastrophically* so for force — R² falls to −9.

## 1. What "tactile" means here

Tactile is **vision-based tactile sensing (VBTS)**: an internal camera imaging a deformable skin. The paper's framing device is a three-way taxonomy of transduction principle, which is the axis it varies:

- **IMM** (Intensity Mapping) — shape/pressure from reflected-light variation
- **MDM** (Marker Displacement) — deformation from tracked printed/embedded markers
- **MFM** (Modality Fusion) — transparent "see-through" skins fusing visual appearance with tactile cues

Seven sensors, deliberately spanning families and hybrids:

| Sensor | Principle | Images |
|---|---|---|
| GelSightNoMarker | IMM | 16,917 |
| GelSightMarker | IMM+MDM | 15,487 |
| MagicGripper | IMM+MDM+MFM | 16,892 |
| MagicTac | IMM+MDM+MFM | 12,496 |
| TacTip | MDM | 11,000 |
| ViTac | IMM+MFM | 11,000 |
| ViTacTip | MDM+MFM | 23,008 |

The argument is that these differ in optical design, illumination, gel structure, marker configuration, spatial resolution and contact geometry — so a model trained on one may be capturing sensor artifacts rather than contact information, and nothing in the existing literature separates the two.

## 2. Data curriculum

**106,800 labelled tactile images**, organised by *task* rather than by sensor, with shared label spaces across sensors so that sensor variation is isolated from annotation mismatch.

- **Shape classification** (30,094 images) — 3D-printed indenters of diverse geometry pressed against each sensor; 9-way over categories shared by all seven.
- **Grating classification** (40,509 images) — 3D-printed line- and dot-like gratings, widths/spacings **0.3–1.75 mm in 0.05 mm increments**; 100 presses per sample with varying TCP yaw. 30 classes per configuration.
- **Force regression** (36,197 images) — an M3813B six-axis F/T transducer as ground truth, with a 2 mm-radius spherical indenter. The robot approaches in **0.1 mm** steps until contact, then applies small x–y displacements to capture both normal and shear response.

Positioned against FoTa (3.1M, 13 sensors), TacQuad (72.6k), TacBench (460k): TacVerse is smaller but is the only one whose stated objective is *quantifying sensor shift* under controlled task settings rather than learning a representation.

## 3. Model

Deliberately unremarkable — the benchmark is architecture-controlled. ResNet-18 and ViT under four initialisations (random, ImageNet, MAE-pretrained on TacVerse itself).

## 4. How tactile enters the model

Tactile *is* the input; there is no fusion problem here. The interesting encoding question the paper answers is which **initialisation** yields a sensor-robust tactile feature — and the answer is masked autoencoding on tactile images themselves.

## 5. Experiment setup

Three protocols, which is the contribution:

1. **Within-sensor** — train and test on the same sensor (upper bound / task learnability check).
2. **Zero-shot cross-sensor** — fixed-source one-to-many; train once on a source, evaluate on unseen targets.
3. **Few-shot adaptation** — sweep the ratio of labelled target data for force regression.

Plus a **representation study** across backbones/initialisations, and Grad-CAM for qualitative attention analysis.

## 6. Does tactile actually help — or rather, does it transfer?

**Within-sensor is strong everywhere**, confirming the observations are informative. Then it falls apart.

**Grating classification**, source GelSightMarker: GelSightNoMarker is the most transferable target, and even it drops **0.903 → 0.248**. MagicGripper and ViTacTip are near chance. Grad-CAM shows why: within-sensor the model attends tightly to the contact region and the grating structure; cross-sensor the activation becomes diffuse and *surrounds* rather than concentrates on the target. Sensor mismatch destroys not just accuracy but localisation.

**Force regression**, source GelSightNoMarker:

| Target | Protocol | MAE ↓ | RMSE ↓ | R² ↑ |
|---|---|---|---|---|
| GelSightNoMarker | within | 0.126 | 0.186 | **0.590** |
| GelSightMarker | zero-shot | 0.287 | 0.348 | −0.014 |
| MagicGripper | zero-shot | 0.766 | 0.976 | **−6.046** |
| ViTacTip | zero-shot | 0.863 | 1.277 | **−9.058** |

Negative R² means the transferred model is worse than predicting the mean. This is the single most useful number in the cross-sensor literature: it puts a floor under claims that tactile features are "sensor-agnostic". Shape classification is comparatively robust; texture and force are not.

**Few-shot adaptation** monotonically improves RMSE and R² as target-label ratio rises, but **never reaches the within-sensor upper bound**.

**Representation study** — MAE pretraining is the most consistent winner:

| Task | Metric | ResNet-18 | ViT-Rand | ViT-ImgNet | ViT-MAE |
|---|---|---|---|---|---|
| Grating (GelSightMarker) | Acc ↑ | 0.808 | 0.552 | 0.887 | **0.944** |
| Grating (GelSightNoMarker) | Acc ↑ | 0.711 | 0.576 | 0.768 | **0.938** |
| Force (GelSightMarker) | RMSE ↓ | 0.241 | 0.375 | 0.197 | **0.158** |
| Force (GelSightNoMarker) | RMSE ↓ | 0.258 | 0.335 | 0.186 | **0.170** |

MAE wins force regression on **all four** sensors and grating on three of four. Shape splits between ImageNet and MAE — geometric structure is apparently generic enough that visual priors suffice, while texture and force need tactile-specific pretraining.

**Honest limits, as stated.** Controlled laboratory interactions only; grating and force use fixed-source evaluation; no comparison against specialised tactile foundation models — so this quantifies the problem that [[ftp-1]], [[anytouch2]], [[htt]] and [[tactx]] claim to solve, without testing whether they solve it.

## 7. What it adds that the others don't

An **architecture-controlled** measurement of sensor shift with shared label spaces — the negative-R² result is the empirical justification for the entire cross-sensor foundation-model line, and simultaneously the yardstick none of those papers is evaluated against.
