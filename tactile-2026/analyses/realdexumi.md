# RealDexUMI — A Wearable Universal Manipulation Interface for Dexterous Robot Learning

**arXiv:2606.06033** (v2) · Peking University + **BeingBeyond** + Beihang + LinkerBot + Tsinghua (C. Xu, Y. Jiang, Huan, Fu, H. Zhou, W. Yuan, J. Yu, W. Zhang, H. Yuan, Z. Lu) · Jun 2026 · [site](https://research.beingbeyond.com/realdexumi)

**One line.** Makes **the deployed robot hand itself the demonstration device** — the human wears it — so tactile signals, in-hand views, contacts and hand actions are *identical* between collection and deployment. Zero retargeting.

## 1. The concept it introduces — *deployable dexterity*

> *"the relevant quantity is therefore not captured dexterity alone, but **deployable dexterity**: the extent to which demonstrated hand actions remain executable by the deployed dexterous end effector while the associated contacts, tactile signals, and observations are preserved."*

Both existing families fail it. Hand-motion interfaces *"rely on human-to-robot mappings across different kinematics, contact geometries, and sensing channels"*, and *"such offline or online retargeting can distort contact-rich interactions."* Robot-specific leader–follower teleoperation removes mapping ambiguity but *"ties data collection to robot-specific hardware and does not naturally scale."*

## 2. System

A **shared dexterous end-effector module** serving as both the wearable collection interface and the deployed robot hand:
- lightweight dexterous hand (**6 active DoF**)
- **in-hand vision**
- **fingertip tactile sensing**

Plus a **palm-side isomorphic teleoperation glove** mapping human finger inputs directly to robot-hand joint commands — *"real-time, retargeting-free, intuitive, and precise."*

Result: **zero-gap end-effector data**, with matched in-hand observations, tactile signals, contacts and hand actions between collection and deployment.

A **relative end-effector action representation** lets the same policy run across embodiments *"by changing only the inverse kinematics"* and low-level controller.

## 3. Results

**88.75% average success across eight real-robot tasks** spanning fine-grained, contact-rich, long-horizon and bimanual manipulation. Generalises to unseen initial poses. Transfers across **three embodiments**.

## 4. Stated limitations, both honest

- **Observation is mainly local** (in-hand vision), which *"limits tasks that require object search, long-range planning, or explicit task-progress reasoning."* Adding egocentric or global views is proposed, but *"keeping such views aligned between wearable collection and robot deployment remains challenging"* — the very alignment the design is built on works against adding a global view.
- **A dexterity/weight/controllability trade-off**: *"The current hand with six active DoFs keeps the system lightweight and enables intuitive isomorphic glove control, but it does not cover the full capability of higher-DoF dexterous hands."*

## 5. What it adds that the others don't

**Eliminating the embodiment gap by sharing the hardware.** [[twins]] does this for the whole arm-and-torso by building an isomorphic robot; RealDexUMI does it for the end-effector by having the human *wear* the robot hand, which is cheaper and scales to in-the-wild collection. Because the tactile sensors are literally the same sensors, it is the only interface in this survey where collection-time and deployment-time tactile distributions are identical by construction — sidestepping the cross-sensor problem that [[htt]], [[tactx]] and [[uniforce]] all exist to solve.
