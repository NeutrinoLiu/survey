# MiTaS — Multi-Resolution Tactile Imitation Learning for Contact-Rich Robotic Manipulation

**arXiv:2606.06281** · TU Darmstadt + Hessian AI + Robotics Institute Germany (Krohn, Helmut, Funk, Peters, Prasad, Chalvatzaki) · Jun 2026 · [site](http://mitas-touch.github.io)

**One line.** Argues that **spatial tactile and high-frequency tactile are different modalities**, not one — and fuses a GelSight Mini with an **event-based Evetac** sensor at their native temporal resolutions.

## 1. What "tactile" means here — two sensors, two timescales

Grounded in human physiology: *"Humans rely on multi-modal touch sensing... seamlessly integrating information from diverse mechanoreceptors operating at different temporal and spatial resolutions."*

| Sensor | Provides | Misses |
|---|---|---|
| **GelSight Mini** (frame-based) | high-resolution local contact geometry, deformation, surface structure | *"less suited to capturing very fast contact changes, such as impact, incipient slip, high-frequency vibration"* — these *"may occur between frames or be smoothed out by conventional image-based processing"* |
| **Evetac** (event-based) | high temporal resolution, rapid contact dynamics | spatial detail |
| **RGB camera** | global scene context | contact entirely |

## 2. Model

**Modality-specific convolutional stems** + **transformer-based fusion**, preserving each stream's native characteristics before integration, conditioning a **flow-matching policy**.

The architectural claim is that fusing at native resolution beats resampling everything to a common rate — collapse the streams first and the event sensor's advantage disappears.

## 3. Results

Five contact-rich tasks (including key insertion):

| Policy | Average success |
|---|---|
| Vision-only | **31%** |
| Visual + tactile (GelSight only) | 54% |
| **MiTaS (RGB + GelSight + Evetac)** | **80%** |

Vision-only *"fail[s] completely in two tasks."* The 54% → 80% step is the paper's contribution: it is not tactile that buys the last 26 points, it is *multi-resolution* tactile.

**The co-training result is the practically important one.** Co-training a visuo-tactile model with multi-tactile data **boosts performance by over 10% on certain tasks — without access to the Evetac sensor at policy evaluation.** An expensive event-based sensor used only at training time improves a policy deployed with a common GelSight. That is the same "tactile as training-time signal" pattern as [[hapticvla]] and [[unitacvla]], here across sensor *types* rather than presence/absence.

**Attention analysis** across task execution reveals which sensor matters when, *"validating our multi-resolution tactile sensing approach"* — the empirical version of the paper's premise.

## 4. What it adds that the others don't

**Heterogeneous tactile fusion within one gripper.** The cross-sensor cluster ([[htt]], [[tactx]], [[ftp-1]]) unifies *different sensors on different robots* so one model serves all; MiTaS uses *different sensors simultaneously on the same gripper* because they measure different physics. And the co-training result offers a resolution to the hardware-cost problem: instrument the data-collection rig richly, deploy cheaply — the sensor-type analogue of [[hapticvla]]'s argument.
