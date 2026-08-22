# TaCo — A Benchmark for Lossless and Lossy Codecs of Heterogeneous Tactile Data

**arXiv:2602.09893** · **ICLR 2026** · Shanghai Jiao Tong University + Paxini Tech (Cheng, Zhao, Wang, Zhang, Song) · Feb 2026

**One line.** The only work in this survey that treats tactile as a *bitrate* problem — and shows a 20–200× reduction costs almost nothing in classification or grasping success.

## 1. What "tactile" means here

Deliberately heterogeneous, spanning both sensing families:
- **Visuo-tactile images** — GelSight/DIGIT-style surface deformation (Touch and Go, SSVTP, YCB-Slide, ObjectFolder)
- **Force arrays** — Paxini force-based sensors (ObjTac)

The framing quantity is **bandwidth**. Raw Touch-and-Go tactile video is ~**200 Mbps** (24 bit/px × 480 × 640 × 30 fps). The paper's motivating scenario is the MCU-to-motherboard link inside a dexterous hand, plus teleoperation latency and Open-X-scale storage (1M+ trajectories, 8,965 GB). Their analogy is the cortical homunculus: hands claim a disproportionate share of the sensing budget, and low-cost embedded MCUs cannot carry it.

The technical difficulty they name is exactly the one the foundation-model papers face from the other direction: **heterogeneity of sensing principle** means one codec must handle deformation images and force vectors alike.

## 2. Data curriculum

**Five datasets, >250K frames**: Touch and Go, ObjectFolder, YCB-Slide, SSVTP, ObjTac.

Split discipline: 70% of two datasets train the proposed codecs; the remaining 30% plus **all** of SSVTP, YCB-Slide and ObjTac are held out for evaluation of every method — so three datasets are fully out-of-distribution for the trained codecs.

**30 codecs** evaluated:
- **17 off-the-shelf** — GZIP, BZIP2, ZSTD, PNG, WebP, FLIF, BPG, JPEG-XL, JPEG2000, x265, SVT-AV1, VVenC, HM-Intra, HM-SCC, VTM-Intra, VTM-SCC
- **13 neural** — DLPR, LALIC, TCM, P2LLM, ELIC, LMIC (RWKV / Llama3), DualComp-I, DCVC-FM/DC/RT
- 14 support lossless; most neural ones are applied **without tactile-specific adaptation** to test cross-domain generalisation.

## 3. Model — TaCo-LL and TaCo-L

The first codecs *trained on tactile data*.

- **TaCo-LL** (lossless) — built on DualComp-I; input is divided and tokenised (Fig. 3), trained with FusedAdam, cosine-annealed LR 1e-4 → 2e-5 over 20 epochs. Variants at **12M / 48M / 96M** parameters.
- **TaCo-L** (lossy) — neural image-compression architecture retrained on tactile data, Adam.

## 4. How tactile enters the model

As the payload, not as a conditioning signal. The interesting observation is a *representational* one: force-based ObjTac data has statistics **similar to screen content** — large uniform regions and repetitive patterns — which is why screen-content-optimised video codecs (VTM-SCC, HM-SCC) reach BD-Rates of −44.3% and −44.5% on it. That is a genuinely useful fact about the structure of taxel-array data.

## 5. Experiment setup

Four task families, spanning human, machine and robot consumers:

1. **Lossless compression** → storage. Metric: bits/Byte.
2. **Lossy compression for human visualisation** → PSNR vs. bpp, BD-Rate (uncompressed = 24 bpp).
3. **Semantic classification** → material/object top-1 accuracy under SVM and linear probe, fixed 60/40 split.
4. **Dexterous robotic grasping** → success rate of lifting (`s_lift`, object stable at 0.1 m) plus a second success criterion, in simulation and on real hardware (ice tea, box).

## 6. Does compression hurt?

**Lossless.** TaCo-LL-96M is best on all five datasets, reaching **0.447 bits/Byte** — roughly 18× — and the TaCo-LL family is competitive with far larger LLM-based compressors (P2LLM, Llama3-8B, RWKV-7B) at a fraction of the parameters.

**Lossy rate–distortion.** TaCo-L wins on all five datasets, with BD-Rate reductions of **−61.8% (Touch and Go), −24.3% (ObjectFolder), −27.4% (YCB-Slide), −27.0% (ObjTac)**.

**Classification is remarkably robust to compression.** From 24 bpp down to **0.118 bpp** (~200×):

| Dataset | Uncompressed | TaCo-L compressed |
|---|---|---|
| Touch and Go (SVM) | 76.63% | 75.12% |
| Touch and Go (lin. reg.) | — | 71.89% |
| ObjectFolder (SVM) | 44.11% | 43.08% |
| YCB-Slide | — | 98.01 / 98.20% |

A ~1.5-point accuracy cost for a ~200× bitrate reduction. This is the most consequential number in the paper: **the semantic content of tactile signals lives in a tiny fraction of their bits.**

**Grasping.** All methods compress 24 bpp down to 0.025–0.5 bpp with "only a moderate decrement in grasping success rate"; TaCo-L outperforms JPEG-XL and LALIC. Reported operating points from the summary figure: <0.1 bpp at ~40 dB for human viewing, 0.1 bpp at ~95% accuracy for classification, 0.05 bpp at ~60% success for grasping. Note the grasping number degrades much more than classification — closed-loop control is the harder consumer, which makes sense given it depends on fine temporal force detail rather than category-level texture.

**Limits.** Everything is frame-wise image compression; the authors state that a proper **video-like tactile codec** (retraining DCVC-series neural video models on tactile) is future work — i.e. inter-frame temporal redundancy, likely the largest remaining win, is not yet exploited by the proposed models.

## 7. What it adds that the others don't

It reframes tactile from a modelling problem to a **systems** problem, and produces the number every tactile-hardware and dataset paper should cite: touch is ~200× more compressible than its raw form for semantic purposes. That both bounds the information content the encoders in [[ftp-1]], [[anytouch2]] and [[htt]] can possibly be extracting, and explains why sparse binary-contact policies like [[roto2]] work at all.
