# Toward Geometry-Scalable Whole-Body Touch for Humanoids — A 3D-Printed Conformal EIT Skin

**arXiv:2608.02080** · Czech Technical University in Prague + University of Colorado Boulder + KAIST (H. Chen, Kohlbrenner, Kubik, Rustler, Dickhans, Bartunek, Roncone, H. Lee, **Hoffmann**) · Aug 2026

**One line.** Attacks whole-body tactile from the manufacturing side: a **continuous** EIT sensing domain that can be 3D-printed onto arbitrary robot curvature, replacing dense taxel arrays that *"scale poorly with surface area, wiring complexity, and robot-specific curvature."*

## 1. The scaling problem

Large-area tactile skins *"commonly rely on dense or modular arrays of discrete sensing cells, requiring extensive interconnection, packaging, and **geometry-specific integration**."* Every new robot morphology means a redesign.

**EIT** reconstructs contact-induced conductivity changes across a **continuous** sensing domain from a **limited number of boundary electrodes** — decoupling sensing area from wiring count. But applying it to *"non-planar, curved, and robot-specific surfaces remains challenging."*

## 2. Design

- **Flexible conductive TPU layer** forming a continuous sensing domain
- **Contact-induced coupling with conductive patches** producing boundary voltage changes
- **One-step Gauss–Newton EIT solver** for reconstruction
- **Geometry-adaptable additive-manufacturing workflow**

Two electromechanical findings from the design-space characterisation: **low-resistance contact-enhancement patches** and a **porous conductive TPU sensing layer** improve sensitivity *"while preserving printability"* — sensitivity and manufacturability traded explicitly.

## 3. Results

Validated on three geometries of increasing difficulty: **planar**, **curved U-shaped**, and a qualitative **iCub-face-shaped** prototype.

The curved sensor achieves **6 mm mean localisation error over 18 contact positions — without supervised post-processing.** No learned correction, which matters: a purely physical reconstruction that generalises to new geometry is what "geometry-scalable" requires.

## 4. What it adds that the others don't

**Manufacturability as the design objective.** [[twins]] and [[wt-umi]] both need body-surface tactile and both are limited by what can be instrumented — WT-UMI states plainly that only palms, forearms and chest are covered. This is the hardware answer: print the skin to fit the robot. The complementarity with the learning literature is direct — [[tactilegenesis]] shows in simulation that **coverage dominates resolution**, and 6 mm localisation over a continuous printed surface is exactly the coverage/resolution trade that finding endorses.
