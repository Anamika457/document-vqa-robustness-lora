# Notebook 2: Scaled Evaluation with a Held-Out Test Set

Extends Notebook 1 with a larger training set and a proper three-way data split, so
that reported results are less sensitive to sampling noise and reflect standard
train/validation/test discipline.

## Setup

- **Model and LoRA config:** unchanged from Notebook 1
- **Data:** 300 training examples, 60 validation examples, 60 held-out test examples,
  drawn from the combined train and test splits of `nielsr/docvqa_1200_examples`
- **Scoring:** both exact match and fuzzy match are reported (see Methodology below)
- **Corruption functions:** blur and rotation are applied after resizing images to the
  model's input resolution. The original implementation applied a fixed-radius blur to
  the full-resolution source image, which was then substantially smoothed away by the
  model's own downscaling step, understating the intended corruption severity. This was
  corrected, and all results below reflect the corrected version.

## Results — held-out test set (n=60)

| Condition | Exact Match | Fuzzy Match |
|---|---|---|
| Base model, clean | 0.117 | 0.183 |
| Fine-tuned, clean | 0.217 | 0.267 |
| Fine-tuned, blurred | 0.267 | 0.450 |
| Fine-tuned, rotated | 0.083 | 0.217 |

Validation set (n=60, fuzzy match): clean 0.30, blurred 0.40, rotated 0.20.

## Methodology: exact and fuzzy scoring

An initial fuzzy-match-only pass showed blurred images scoring higher than clean
images — a counterintuitive result that warranted scrutiny before being reported. Two
checks were run. First, predictions under blur were compared directly against clean
predictions for the same questions; blurred predictions were not shorter or more
generic on average (13.6 vs. 12.0 characters), which ruled out the fuzzy metric
rewarding vague guesses. Second, after the corruption-resolution bug above was found
and fixed, the effect persisted rather than disappearing. Exact-match scoring, which
cannot be inflated by partial or generic answers, shows the same direction (0.267 under
blur vs. 0.217 on clean).

## Discussion

Rotation consistently reduced accuracy, holding across both the validation set (0.20
vs. 0.30 clean) and the test set (0.083 vs. 0.217 clean, exact match) — this is the
most consistent result in the project so far.

Blur did not show the expected degradation and instead showed a small, repeated
advantage over clean images, holding across both validation and test sets and
surviving the correction described above. This is reported as a genuine open finding
rather than dismissed as noise, though at n=60 it isn't confirmed and would benefit
from a larger sample or repeated trials.

Fine-tuning produced a modest improvement in clean test accuracy (0.117 to 0.217, exact
match), though any real effect at this training scale is small relative to the noise
introduced by a 60-example test set.