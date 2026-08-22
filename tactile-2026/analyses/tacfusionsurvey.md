# Tactile-based Multimodal Fusion in Embodied Intelligence — A Survey

**arXiv:2605.17336** · XJTU + HKUST(GZ) + Nanjing + Purple Mountain Lab + Wuhan + James Cook + BUPT + Fudan + Linkerbot + Swinburne (Cao, Tian, Guan, Mu, Sun, S. Liang, D. Liu, T. Huang, Yue, Ding, Fang, A. Zhou, Han, **Xiong**) · May 2026 · [collection](https://github.com/Wayne-coding/Multimodal-Tactile-Fusion)

**Coverage:** multimodal tactile fusion research **up to Q1 2026** — the first dedicated survey on the topic.

## The problem it names

*"Unimodal tactile perception is inherently limited by its **sparse spatial coverage and lack of global semantic context**."* Hence fusion with vision and language.

And the state of the field: *"existing researches remain **fragmented across disparate datasets, sensing modalities, and tasks, lacking a unified theoretical framework**."*

## The taxonomy

**Data axis** — four dataset families:
- Tactile–Vision
- Tactile–Language
- Tactile–Vision–Language
- Tactile–Vision–Other (audio, action streams)

**Method axis** — three pillars:
1. **Multimodal Perception and Recognition** — object understanding, grasp prediction
2. **Cross-Modal Generation** — bidirectional translation between tactile, vision and text
3. **Multimodal Interaction** — feedback control, language-guided manipulation

Plus representative sensing hardware, evaluation metrics and benchmark settings.

## The observation on evaluation

The section on metrics contains the survey's most useful judgement: *"most of the studies adopt evaluation protocols **tailored to specific tasks and application settings rather than following a common benchmarking pipeline**. Understanding these evaluation settings is therefore important for interpreting reported results."*

Metrics are grouped into **classification-based, similarity-based and ranking-based** measures, with the caveat that *"no single metric is suitable for all scenarios."*

That fragmentation is exactly what [[rct]] exploits to show retrieval scores are inflated, what [[tacverse]] addresses with shared label spaces, and what [[softvtbench]] tries to fix with an evaluator-only channel — three 2026 papers independently attacking the problem this survey documents.

## Position in the survey

Useful as a **map of the pre-2026 landscape** and of the tactile-vision-language line specifically. Its blind spots are the ones its Q1-2026 cutoff implies: the world-model cluster ([[vt-wm]] onward), the foundation-policy line ([[ftp-1]], [[n0-twam]]), and the evaluation-methodology corrections ([[rct]], [[softvtbench]]) all postdate it. Read together with [[vbtintel]] (hardware-to-policy pipeline) and [[physintsurvey]] (force-aware learning architectures), the three surveys partition the field along data/hardware/architecture axes with surprisingly little overlap.
