# Classifying "made of / from / with" — DistilBERT fine-tuning

**English** | [한국어](README.ko.md)

A **text classifier** that predicts the right preposition in the construction
*"X is made **of / from / with** Y"* from the sentence alone. DistilBERT is fine-tuned
with **PyTorch + Hugging Face Transformers**.

> This project grows out of my master's thesis, *"Distinguishing 'Made of/from/with' Based on
> the Semantic Relations of the Arguments"* (Heejeong Wee, UNC Chapel Hill, 2024). In the thesis,
> **GPT-4o (zero-shot)** reached only **39%** overall accuracy on this task, and **0%** on `from`.
> This project tests directly: *how much better does a model do when it is trained on the data?*

---

## Results (test set, 141 sentences)

| Model | Accuracy | **Macro-F1** |
|------|:------:|:------------:|
| ① Majority class (always `of`) | 0.546 | 0.235 |
| ② TF-IDF + logistic regression | 0.454 | 0.379 |
| **③ DistilBERT (fine-tuned)** | **0.610** | **0.575** |

DistilBERT beats both baselines on **both accuracy and Macro-F1**.

**DistilBERT per-class scores (test set)**

| Preposition | precision | recall | F1 |
|--------|:---:|:---:|:---:|
| of | 0.785 | 0.662 | 0.718 |
| from | 0.444 | 0.432 | **0.438** |
| with | 0.475 | 0.704 | 0.567 |

- Compared with **0%** on `from` in the thesis, the trained model reaches a `from` F1 of **0.438**.
- Detailed results: [`results/`](./results) (score table, confusion matrix, per-class report)

> Note: the thesis's 39% used a different data split and input format, so it is not a one-to-one
> comparison; it is a motivating number that shows the effect of training.

---

## Task

- **Input:** the sentence with the answer preposition masked (`made of/from/with` → `made ___`)
- **Output:** one of `of` / `from` / `with` (3-way classification)
- **No answer leakage:** the preposition is masked in the input so the model cannot copy it.

## Data

- Source: **Appendix 2** of the thesis above (original sentences extracted from COCA, 2016–2017).
- Size: **939 sentences** — of 511 / from 248 / with 180 (imbalanced classes).
- Built by coordinate-based parsing of the PDF appendix with `pdfplumber`. Details: [`data/README.md`](./data/README.md)
- Split: train 657 / validation 141 / test 141 (stratified to keep class ratios).
- **The data files are not included in this repository.** Given COCA's terms of use, the CSV is kept local. How to build it and its format are in [`data/README.md`](./data/README.md).

## Method (pipeline)

1. **Data split** — train/val/test, stratified → `notebooks/02_data_split.ipynb`
2. **Baselines** — majority class + TF-IDF with logistic regression → `notebooks/03_baseline.ipynb`
3. **Fine-tuning** — DistilBERT with a class-weighted loss (for imbalance), 5 epochs, MPS (Apple M4) acceleration
   → `notebooks/04_finetune_distilbert.ipynb`
4. **Final evaluation** — compare the three models on the test set → `notebooks/05_evaluate_test.ipynb`

Metrics are **accuracy** and **Macro-F1** (which weights minority classes equally, the key metric under imbalance).

---

## Reproducing

```bash
# 1) Create and activate the conda environment
conda env create -f environment.yml
conda activate default

# 2) Prepare data/made_prepositions.csv (see data/README.md)

# 3) Run the notebooks in order (Jupyter or VS Code)
#    02 → 03 → 04 → 05
jupyter lab
```

> On Apple Silicon (M-series), PyTorch uses MPS acceleration automatically.
> Trained models are not committed because of their size (`outputs/` is ignored); running
> notebook 04 regenerates them.

## Repository layout

```
data/
  README.md                  # Data card (source, columns, distribution, how to build)
  made_prepositions.csv      # Full dataset, 939 rows (local only, not in git)
  splits/                    # train/val/test splits (created by notebook 02, not in git)
notebooks/
  02_data_split.ipynb        # Data split
  03_baseline.ipynb          # Baselines
  04_finetune_distilbert.ipynb  # DistilBERT fine-tuning (core)
  05_evaluate_test.ipynb     # Final test-set evaluation
results/                     # Final scores, confusion matrix, report
environment.yml              # conda environment
```

Every notebook is annotated line by line (comments in Korean), so the code can be followed from start to finish.

## Limitations and next steps

- With only 939 examples there is a risk of overfitting, and room to improve the scores.
- As the thesis concludes, preposition choice **may not be deterministic** (several answers can be
  acceptable depending on pragmatics or emphasis), which caps achievable accuracy.
- Next: a larger backbone (BERT-large) or LoRA-based LLM fine-tuning, comparing inputs that also
  supply product/material, and cross-validation to assess stability.
