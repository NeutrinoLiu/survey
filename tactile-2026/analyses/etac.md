# ETac — A Lightweight and Efficient Tactile Simulation Framework for Learning Dexterous Manipulation

**arXiv:2604.20295** · ShanghaiTech University (Z. Xu, F. Zhao, X. Huang, C. Xiao) · Apr 2026 · [site](https://lassford.github.io/ETac/)

**One line.** Replaces FEM with a **learned deformation-propagation model** — keeping FEM-comparable surface deformation while running **4,096 parallel environments on a single RTX 4090**.

## 1. The trade-off it targets

*"Learning tactile-based skills through RL remains challenging, primarily due to the lack of tactile simulators that are both high-fidelity and computationally efficient."*

FEM-based simulators *"capture high-quality contact deformations"* but *"are developed as standalone tools, [and] their integration with multi-fingered robotic hands remains"* difficult — which is the practical obstacle for whole-hand tactile RL.

## 2. Method

A **lightweight data-driven deformation propagation model** for soft-body contact dynamics, replacing the solver rather than approximating the geometry. Deformation estimates are *"comparable to FEM"* and the model *"demonstrates applicability for modeling real tactile sensors."*

## 3. Results

| | |
|---|---|
| Parallel environments | **4,096** on a single RTX 4090 |
| Throughput | **869 FPS** total |
| Blind grasping policy | **84.45%** average success across four object types |

The demonstration task is well chosen: a **blind** grasping policy using **large-area tactile feedback** to manipulate diverse objects — no vision, so the tactile simulation quality is the only thing determining success, and whole-hand coverage is what the simulator's efficiency buys.

## 4. What it adds that the others don't

**Large-area tactile at RL scale.** The efficiency/fidelity position sits between [[tacauchy]] (rigorous FEM, 60 envs) and [[tactilegenesis]] (abstract sensor models, 20,000 envs) — a *learned* surrogate for the soft-body solver that keeps deformation realism where it matters. Its blind-grasping result also supports [[tactilegenesis]]'s independent finding that **whole-hand coverage** is the variable that matters most for dexterous tactile policies, arrived at here by building the simulator that makes covering the whole hand computationally affordable.
