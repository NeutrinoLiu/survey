# RGB-S — Image-Aligned Tactile Saliency for Robust Dexterous Manipulation

**arXiv:2606.08765** (v2) · ShanghaiTech + BIGAI (S. Luo, K. Wu, X. Zhou, W. Li, Jiao, Xiao) · Jun 2026 · [site](https://touch-as-saliency.github.io)

**One line.** Projects tactile contacts **into the camera image** as force-modulated Gaussian blobs, using forward kinematics and calibration — so the policy never has to learn where a touch happened in space.

## 1. The modality asymmetry

*"Tactile and proprioceptive signals are typically sparse, low-dimensional, and robot-specific. Due to heterogeneous hardware, they lack the standardized datasets and transferable pretraining pipelines that have driven progress in vision. This creates a fundamental **modality asymmetry**: strong visual priors are readily available, whereas tactile priors are not."*

And the consequence for fusion: classical implicit multimodal embeddings *"often lose spatial correspondence under occlusion"* — spatial disorder. Policies *"must learn cross-modal correspondences implicitly from limited demonstrations, without leveraging geometric priors"*, making them data-inefficient and fragile.

But the correspondence is **not** something that needs learning: the robot knows its own kinematics.

## 2. Method

1. **Forward kinematics** maps 1-D/2-D tactile sensor readings to **3D locations** via the URDF
2. **Camera calibration** projects those locations onto the **RGB image plane**
3. Render **force-modulated Gaussian saliency maps** — force magnitude sets intensity, Gaussian spread models *"spatial uncertainty arising from kinematic and calibration errors"*
4. Integrate through a **zero-initialised conditioning architecture**, which *"injects physical contact priors into standard visual backbones while **preserving pre-trained visual representations**"*

The zero-init is what makes this safe: the model starts exactly as the pretrained visual policy and learns to use the saliency channel, so nothing is forgotten — the concern [[at-vla]] and [[waketouch]] address by other means.

Modelling calibration error as Gaussian spread rather than a point is a nice touch: the saliency is honest about how well the projection is known.

## 3. Results

Six dexterous manipulation tasks in simulation and the real world **under severe visual occlusion**: **+26.7 percentage points** real-world occluded success over the strongest **implicit** visuo-tactile baseline.

The comparison is the right one — against implicit fusion with the same tactile signal — so the gain is attributable to the *explicit spatial grounding*, not to tactile availability.

## 4. What it adds that the others don't

**Using the robot's own geometry as free supervision.** Where every cross-modal alignment method in this survey *learns* the correspondence between touch and scene — contrastively ([[tactx]]), by physical units ([[uniforce]]), by paired data ([[htt]]) — RGB-S computes it from the URDF and the camera extrinsics, which are already known. It is also, together with [[tap-vla]], one of two independent 2026 works arriving at the same idea from different directions: **render touch into the visual observation space the pretrained model already understands**, rather than adding a stream it must learn to read.
