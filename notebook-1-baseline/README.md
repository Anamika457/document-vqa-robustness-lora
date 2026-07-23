# Notebook 1: Baseline Fine-tuning Pipeline

Fine-tunes SmolVLM-256M-Instruct on document VQA using LoRA and evaluates accuracy on
clean, blurred, and rotated document images. This notebook establishes the core
pipeline used across the project: data loading, LoRA setup, training, and evaluation.

## Setup

- **Model:** SmolVLM-256M-Instruct
- **Data:** 100 training examples, 25 validation examples, sampled from
  `nielsr/docvqa_1200_examples`
- **Fine-tuning:** LoRA (r=8, alpha=16) applied to the attention projection layers
  (`q_proj`, `k_proj`, `v_proj`, `o_proj`), base model weights frozen
- **Scoring:** substring and numeric-tolerant fuzzy matching against ground truth

## Results

| Condition | Accuracy |
|---|---|
| Base model, clean images | 0.16 |
| Fine-tuned, clean images | 0.12 |
| Fine-tuned, blurred images | 0.20 |
| Fine-tuned, rotated images | 0.20 |

## Discussion

At this data scale, fine-tuning did not produce a measurable improvement in clean-set
accuracy — the observed difference amounts to a single example on a 25-example set and
isn't meaningful on its own. Blur and rotation did not appear to reduce accuracy in this
run, but with only 25 validation examples this result is inconclusive rather than
evidence of genuine robustness.

Notebook 2 revisits this comparison with a larger, held-out test set and both exact and
fuzzy scoring. Notebook 3 later identified that the corruption functions used here
applied blur before resizing to the model's input resolution, which weakens their
effect substantially — this was corrected in later notebooks, and the corrected results
should be treated as the more reliable version of this finding.

## Why this approach

LoRA freezes the pretrained base model and trains a small set of low-rank adapter
matrices instead of updating all weights directly. With only 100 training examples,
full fine-tuning would risk overwriting the model's existing visual and language
understanding; LoRA constrains training to a much smaller set of parameters, reducing
that risk while still allowing the model to adapt.

Evaluating on clean images alone isn't sufficient for a document-verification use case,
since real-world captures are rarely pristine — this motivates testing under blur and
rotation from the outset.