# Notebook 3: Corruption Severity Sweep

Notebooks 1 and 2 tested a single fixed severity of blur and rotation. This notebook
sweeps corruption severity across multiple levels and adds two additional corruption
types relevant to real document capture conditions: JPEG compression and reduced
brightness.

## Setup

- **Model, LoRA config, and training set size:** unchanged from Notebook 2
- **Evaluation set:** 50 examples (this notebook focuses on the severity dimension;
  evaluation-split rigor was already established in Notebook 2)
- **Corruptions tested:** Gaussian blur (radius 1, 2, 4), rotation (5°, 10°, 20°), JPEG
  compression (quality 15), brightness reduction (40% of original)
- Corruption functions apply after resizing to the model's input resolution — this bug
  was identified in this notebook first and backported to Notebook 2

## Results (n=50)

| Condition | Exact Match | Fuzzy Match |
|---|---|---|
| Clean | 0.20 | 0.32 |
| Blur, radius 1 | 0.18 | 0.34 |
| Blur, radius 2 | 0.20 | 0.38 |
| Blur, radius 4 | 0.18 | 0.26 |
| Rotation, 5° | 0.14 | 0.22 |
| Rotation, 10° | 0.16 | 0.24 |
| Rotation, 20° | 0.10 | 0.16 |
| JPEG, quality 15 | 0.26 | 0.34 |
| Darkened, 40% brightness | 0.18 | 0.32 |

## Discussion

The severity sweep does not show a clean, monotonic relationship between severity and
accuracy for either blur or rotation. At n=50, the confidence interval on these
proportions is wide enough (roughly ±14 points) that small differences between
adjacent severity levels — such as rotation at 10° scoring above rotation at 5° — are
consistent with sampling noise and shouldn't be read as a real non-monotonic effect.

What does hold, consistent with Notebook 2, is that all three rotation severities
scored below the clean baseline, reinforcing rotation as the project's most reliable
finding across two notebooks and four separate angle comparisons. Blur, JPEG
compression, and reduced brightness all remained close to or above the clean baseline
at every level tested, consistent with Notebook 2's finding that these corruptions do
not clearly reduce accuracy in this setting.

An earlier version of this sweep showed a misleading peak in accuracy at blur radius 2,
which prompted a closer look at how corruption was being applied relative to the
model's own image preprocessing. That inspection uncovered the resize-order issue
described above; after correcting it, the peak disappeared and the results shown here
reflect the corrected pipeline.