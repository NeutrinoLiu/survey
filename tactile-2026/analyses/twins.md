# TWINS — A Tactile Wearable Isomorphic Arm Networked System for Contact-Rich Manipulation Learning

**arXiv:2608.01733** · **AIST**, Japan (Kitamura, Murooka, Yamanobe, Domae) · Aug 2026 · [site](https://mmurooka.github.io/twins-project-page/)

**One line.** The only work here about **body-surface contact** — carrying an object by embracing it with chest and both arms — and it solves the correspondence problem by making the wearable device and the robot **dimensionally identical**.

## 1. The gap

Humanoids can use *"not only their end effectors but also body surfaces, such as the forearms, upper arms, shoulders, and chest, to support or constrain objects."* But *"most existing systems are primarily designed for teaching end-effector poses and grasping motions. As a result, operators cannot physically interact with objects using a mechanism equivalent to the robot's body."*

And when operator device and robot differ in dimensions or joint configuration, *"additional mapping is required to transfer human motions and contact information"* — introducing exactly the retargeting error that body-surface contact cannot tolerate, since which part of the forearm touches the object determines whether it is supported.

## 2. System

- **Wearable Dual-Arm Device**, worn by the operator
- **Isomorphic Robot** with the **same joint configuration and external dimensions**
- **Distributed tactile sensors embedded in chest and arms**, measuring body-surface contact synchronised with joint motion

Isomorphism is the mechanism: identical dimensions mean zero mapping, so a contact on the operator's left forearm at a given joint configuration corresponds exactly to a contact on the robot's left forearm.

## 3. Results

Demonstrations collected for **four manipulation tasks involving body-surface contact**, imitation policies trained on them, deployed on the Isomorphic Robot — *"enabling manipulation guided by body-surface tactile observations."*

TWINS is presented as a **unified system for demonstration, learning and execution** of this task class.

## 4. Future work, as stated

Mobile manipulation (letting the operator walk during collection); incorporating **human scapular motion** for closer anatomical fidelity; **soft body surfaces** for stability and safety of contact; and **open-hardware release**.

## 5. What it adds that the others don't

**Tactile beyond the fingertips.** Every other entry in this survey instruments the end-effector — gel fingertips, gripper pads, hand taxels. TWINS instruments the *torso and arms*, targeting a class of manipulation (embracing, hooking, bracing) that humanoids can do and grippers cannot, and that is invisible to both cameras and fingertip sensors.

The **isomorphic** design is also the cleanest possible answer to the embodiment gap: rather than retargeting ([[hrdexdb]]), aligning latents ([[tactalign]]), or unifying token spaces ([[ftp-1]]), it removes the gap by construction. That only works when you can build the robot to match the interface — which is precisely the trade the [[realdexumi]] shared-end-effector design makes at a smaller scale. Compare [[wt-umi]], the other whole-body tactile entry, which uses a wearable interface plus force-supervised planning rather than isomorphism.
