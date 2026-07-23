# Document VQA Robustness: LoRA Fine-tuning of SmolVLM-256M

Fine-tunes a small vision-language model (SmolVLM-256M-Instruct) on document visual
question answering using LoRA, then evaluates how robust the resulting model is to
realistic image corruptions — blur, rotation, JPEG compression, and reduced
brightness — rather than reporting clean-set accuracy alone.

The motivation is document and ID verification: systems that read scanned or
photographed documents rarely receive pristine input in practice, so accuracy under
degraded conditions is at least as relevant as accuracy on clean data.

## Notebooks

| Notebook | Focus |
|---|---|
| [1. Baseline](./notebook-1-baseline) | LoRA fine-tuning pipeline and first robustness evaluation |
| [2. Scaled Evaluation](./notebook-2-scaled-eval) | Larger training set, held-out test set, dual exact/fuzzy scoring |
| [3. Severity Sweep](./notebook-3-severity-sweep) | Corruption severity sweep and additional corruption types |

Each notebook folder contains its own README with full methodology and results.

## Findings

**Rotation consistently reduces accuracy.** This held across every evaluation in the
project — validation and test sets in Notebook 2, and three separate rotation angles
in Notebook 3, all in the same direction. Of the findings here, this is the one with
the strongest evidence behind it, and it suggests that in a document-verification
pipeline, correcting image orientation is likely to matter more than deblurring.

**Mild blur, JPEG compression, and reduced brightness do not clearly reduce accuracy,
and blur specifically shows a small, repeated advantage over clean images.** This
result is counterintuitive, so it was checked rather than taken at face value:
predictions under blur were compared directly against clean predictions to rule out
the scoring function rewarding shorter or vaguer answers, and the corruption pipeline
was corrected after an image-resolution issue was found (corruptions were initially
applied before resizing to the model's input size, understating their effect). The
pattern held after both checks. It's reported here as an open finding rather than a
settled conclusion — the evaluation sets used are moderate in size, and a larger sample
would be needed to confirm it with confidence.

**Fine-tuning produced small, inconsistent gains in clean-set accuracy** across the
data scales tested (100–300 training examples). This is broadly consistent with the
limited capacity of a 256M-parameter model to memorize precise, fine-grained facts
(exact dates, percentages, names) from a modest amount of fine-tuning data.

## Model and data

- Model: [SmolVLM-256M-Instruct](https://huggingface.co/HuggingFaceTB/SmolVLM-256M-Instruct)
- Dataset: [nielsr/docvqa_1200_examples](https://huggingface.co/datasets/nielsr/docvqa_1200_examples)
- Fine-tuning: LoRA (`peft`), applied to attention projection layers, base model frozen
- Environment: Kaggle notebooks with GPU acceleration (T4/P100)

## Limitations

Evaluation sets throughout are moderate in size (25–60 examples per condition), so
several reported differences should be read directionally rather than as precise
measurements — this is called out explicitly within each notebook where relevant.
Hyperparameters (LoRA rank, learning rate, epoch count) were set to reasonable defaults
rather than tuned, and experiments were run once rather than averaged over multiple
random seeds, so run-to-run variance is not characterized. The blur/JPEG/brightness
finding, while it survived two independent checks, does not yet have a confirmed
explanation.