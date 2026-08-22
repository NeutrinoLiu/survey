# TransTac — Visuo-Tactile Modality Transition via Ultraviolet-Encoded Transparent Elastomers

**arXiv:2606.04477** · Beijing University of Posts and Telecommunications (L. Yang, **B. Fang**) · Jun 2026 · [code](https://github.com/87361/TransTac)

**One line.** Makes the elastomer **transparent** and encodes the markers in **ultraviolet**, so one device sees through the gel *and* measures its deformation — at a hardware cost of about **$70**.

## 1. The two-sided gap

*"RGB-D cameras provide global depth but **degrade at close range**, while coated VBTS reconstruct only contact deformation."* Opaque elastomer layers *"prevent visual transparency"*, so a conventional gel sensor is blind until it touches.

## 2. Design

- **Transparent elastomer embedded with UV-reflective markers** — invisible in the visible band, so the camera sees the scene through the gel; visible under UV, so the markers can be tracked
- **Binocular** configuration
- **Prior-guided Delaunay stereo matching** for robust sparse triangulation
- A **lightweight detector** for *"densely distributed semitransparent markers"*, giving stable localisation under contact and deformation

## 3. Results

| Result | Value |
|---|---|
| Correspondence robustness vs. global assignment baselines | **+~21%** |
| **Zero-shot semantic recognition on tactile images** | **83.3%** — ~**+50 points** over opaque tactile baselines |
| Class-centre similarity to natural images | **0.2 → >0.77** |
| Hardware cost | **~$70** |

The semantic result is the striking one and the reason it matters beyond hardware. Because the elastomer is transparent, its "tactile" images look like **natural images** — so off-the-shelf vision-language encoders can read them zero-shot at 83.3%, where opaque gel images sit around 33%. The embedding analysis quantifies why: cross-modal alignment with natural images rises from ~0.2 to over 0.77.

That is a hardware solution to the problem the entire tactile-representation cluster attacks in software. [[touch-r1]] notes that *"marker displacement, elastomer deformation, and contact-induced texture patterns are largely absent from natural-image pretraining corpora"*; TransTac makes them present by construction.

Controlled near-distance experiments also quantify **RGB-D depth degradation at close range** and show the extended geometric coverage that visuo-tactile integration provides.

## 4. What it adds that the others don't

**Closing the domain gap optically rather than statistically.** Every tactile foundation model in this survey exists partly because gel images look nothing like the images vision models were pretrained on; TransTac changes the gel. At $70 with open hardware, it is also the cheapest route in this survey to a sensor whose output a pretrained VLM can already interpret — complementary to [[fingereye]], which achieves pre/post-contact continuity with binocular stereo instead of transparency.
