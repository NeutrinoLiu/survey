# Awesome Paper Review

Reading surveys on machine learning systems. One report per subject, each a
stage-by-stage account of how something actually works — read against what the
source material documents, and what it leaves out.

**Live site:** https://neutrinoliu.github.io/survey/

## Reports

| # | Report | Subject | Scope |
|---|--------|---------|-------|
| 01 | [The Curation Ledger](robot-data-curation/) | Pretraining data curation for robotics foundation models | 47 works, 2025–2026 |
| 02 | [What Touch Is Worth](tactile-2026/) | Tactile sensing for robot manipulation — what the modality actually buys | 111 works, 2026 |

## Layout

```
index.html                        # the shelf — links every report
<report-slug>/
  index.html                      # the report, self-contained
  analyses/<work>.md              # per-work source analyses
  bibliography.md                 # annotated bibliography
```

Each report is a single self-contained HTML file: no build step, no bundler, no
external assets beyond Google Fonts. Adding a report means adding a sibling
directory and one card in `index.html`.
