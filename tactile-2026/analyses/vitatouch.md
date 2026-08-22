# VitaTouch — Property-Aware Vision–Tactile–Language Model for Robotic Quality Inspection in Manufacturing

**arXiv:2604.03322** · BUPT + Zhongguancun Academy + BIT (Zong, Q. Jia, Shi, T. Li, J. Li, Lv, G. Chen, Deng) · Apr 2026 · [site](https://vitatouch.github.io/)

**One line.** The only **industrial deployment** paper in this survey: tactile for **quality inspection and defect sorting**, where the failure modes are specular reflection off metal and properties (hardness, roughness, composition) that are not visible at all.

## 1. The industrial case for touch

Industrial inspection is *"still largely dominated by vision-based pipelines"*, which work well under controlled conditions. But *"when inspection is framed as physical understanding rather than appearance recognition, purely visual approaches exhibit inherent limitations."*

Two failure classes:
- **Invisible properties** — *"many manufacturing-critical properties, such as hardness, roughness, and material composition, are not directly observable."*
- **Shop-floor conditions** — *"occlusions, visually inaccessible regions, uneven illumination, and specular reflections on metallic or glossy surfaces."*

VBTS are argued to be the right complement because they *"record contact-induced deformation through RGB tactile readouts, thereby encoding local surface and material properties while being **less susceptible to illumination artifacts** than conventional vision."* On a factory floor with glossy metal parts, that is the decisive property.

## 2. Model

Modality-specific encoders → **dual Q-Former** extracting language-relevant visual and tactile features → compressed into **prefix tokens** for an LLM (BLIP-2 lineage).

Training aligns **each modality with text** *and* **explicitly couples vision and touch** through contrastive learning — a three-way rather than pairwise alignment.

## 3. Data — VitaSet

**186 objects · 52k images · 5.1k human-verified instruction–answer pairs.**

## 4. Results

**General benchmarks:** best on **HCT** and on the overall **TVL** benchmark, competitive on SSVTP.

**VitaSet:**

| Metric | Result |
|---|---|
| Hardness accuracy | **88.89%** |
| Roughness accuracy | **75.13%** |
| Descriptor recall | 54.81% |
| Material-description semantic similarity | peak **0.9009** |

**Defect recognition** with LoRA fine-tuning — the industrial payoff:

| Task | Accuracy |
|---|---|
| 2-category defect | **100.0%** |
| 3-category defect | 96.0% |
| 5-category defect | 92.0% |

**Closed-loop deployment, 100 laboratory robotic trials:** **94.0% recognition accuracy** and **94.0% end-to-end sorting success**.

The gap between the open-vocabulary metrics (descriptor recall 54.81%) and the closed-set defect accuracies (92–100%) is worth noting: **describing** a surface in free language remains hard, while **classifying** it into a fixed defect taxonomy is largely solved. For an industrial deployment, the second is what matters.

## 5. What it adds that the others don't

**A non-manipulation use of touch.** Every other tactile work in this survey uses contact to *control* something; VitaTouch uses it to *inspect* — a setting where the tactile reading is the product, not an intermediate signal. That inverts the usual economics: a factory can afford to instrument one inspection station rather than every robot, and the sensor never has to be fast.

It also makes a specific hardware argument the manipulation literature glosses over: on reflective metal and glossy surfaces — the dominant materials in manufacturing — vision-based tactile is *more* illumination-robust than vision, because its illumination is internal and controlled. That is a real deployment advantage, and one reason [[hapticvla]]'s cost argument does not apply here.
