# RobustTouch — Test-Time Adaptation for Tactile-Vision-Language Models

**arXiv:2602.15873** · Shenzhen Technology University + NYU + Shenzhen University + Tsinghua (C. Ye, H. Jing, Q. Jiang, Lin, Q. Li, X. Tang, J. Jiang) · Jan 2026

**One line.** The only work in this survey about what happens when the **tactile sensor itself degrades in the field** — and it names the right failure mode: **asynchronous** shift, where touch corrupts while vision stays fine.

## 1. The problem

Two observations that no other paper here makes.

**(a) Asynchronous distribution shift.** *"One modality (e.g. touch) may continually degrade due to physical wear while another (e.g. vision) remains stable. Tactile sensors are especially prone to abrupt signal corruption from factors like wear, surface damage, or contact changes."* And crucially: *"simultaneous shifts in all modalities are rare in practice and usually require hardware replacement or environment adjustment."*

This matters because gel sensors are consumables. Every deployment story in this survey assumes the sensor stays calibrated; this one assumes it does not.

**(b) Not all test samples help.** Most multimodal TTA methods operate on *"a fallacious premise: that all test samples are beneficial for adaptation."* Their analysis shows **low-reliability samples where the model attends to irrelevant background introduce noisy gradients and cause up to an 18.5% performance drop**, with error accumulating because existing methods lack dynamic filtering.

Existing fusion is also static: *"averaging or fixed weights, which cannot adapt to the real-time reliability of each modality."*

## 2. Method — reliability, estimated three ways used three ways

Per-modality reliability is estimated from **prediction uncertainty** and **perturbation-based response**, then a single shared signal drives:

1. **Dynamic sample filtering** — exclude noisy/corrupted samples from adaptation
2. **Quality-aware dynamic fusion** — sample-specific adaptive weights per modality, reflecting instantaneous trustworthiness
3. **Reliability-aware loss** — restricts adaptation to high-confidence samples while promoting prediction confidence *and class diversity*, preventing error accumulation and **model collapse**

## 3. Results

On the **TAG-C** benchmark (their contribution — Touch-and-Go with systematic corruptions) and additional TVL scenarios, RobustTouch consistently beats strong TTA baselines (Source, TENT, SAR, READ), with accuracy gains of **up to 49.9% under severe modality corruptions**. Full-model performance in the continuous cross-domain ablation setting: **61.7%** under visual corruption.

## 4. What it adds that the others don't

**Sensor degradation as a first-class problem.** Every deployment result elsewhere in this survey — [[tacmodfusion]]'s dimmed lighting, [[tactile-wam]]'s dim-light insertion, [[n0-twam]]'s visual perturbation — degrades *vision* and shows touch compensating. RobustTouch is the only one that asks what happens when **touch** is the modality that fails, which is the more likely long-run failure given that elastomer gels wear out. The TAG-C benchmark and the asynchronous-shift framing are both reusable, and the observation that **modality reliability must be estimated per-sample at test time** is the natural extension of every gating mechanism in this survey — those gate on *whether contact is happening*, this gates on *whether the sensor can be believed*.
