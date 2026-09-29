# Reproducibility

This file maps each figure-generating script in the repository to its required inputs, randomness, and expected output artifacts.

## Environment

The plotting scripts are written for Python 3 and use standard scientific Python packages including `numpy`, `pandas`, `matplotlib`, `seaborn`, and `scikit-learn` where applicable.

Install the project dependencies before running the figure scripts. All scripts are designed to read results from CSV files rather than embedding experimental result values in the plotting code.

## Figure 1 — ROC curves

**Script:** `fig1.py`

**Purpose:** Computes and plots ROC curves and AUROC values for each task/model pair from model prediction scores.

**Required input schema:**

```text
task,model,y_true,y_score
```

- `task`: task or dataset name.
- `model`: model display name.
- `y_true`: binary ground-truth label (`0` or `1`).
- `y_score`: continuous model score or probability for the positive class.

Optional columns may include `sample_id` and `person_id`.

**Example command:**

```bash
python fig1.py --input predictions.csv --output-dir figures
```

**Random seed:** None. ROC coordinates and AUROC are deterministically computed from the supplied labels and prediction scores.

**Expected outputs:**

```text
figures/<task>_roc.png
figures/<task>_roc.pdf
figures/roc_summary.csv
```

The summary CSV contains `task`, `model`, `auroc`, and `n_samples`.

---

## Figures 2–3 — Efficiency and clinical comparisons

**Script:** `fig2_3.py`

**Purpose:** Generates model-efficiency plots and clinical-comparison plots from tabular result files.

### Efficiency input

**Required columns:**

```text
model
```

At least one of the following metric columns must also be present:

```text
flops,memory,cpu_latency,cpu_throughput
```

Each model should appear once in the efficiency CSV.

**Example command:**

```bash
python fig2_3.py --efficiency-csv efficiency_results.csv --output-dir figures --baseline-model "HiMAE (1.2M)"
```

When `--baseline-model` is provided, relative values are computed by dividing each model's absolute measurement by the selected baseline model's measurement.

### Clinical-comparison input

**Required columns:**

```text
task,model
```

At least one of the following metric columns must also be present:

```text
tp,sensitivity,balanced_accuracy,youden_index
```

Each task/model pair should appear once in the clinical CSV.

**Example command:**

```bash
python fig2_3.py --clinical-csv clinical_results.csv --output-dir figures
```

Both inputs can be provided in the same invocation.

**Random seed:** None. The script performs deterministic transformations and plotting of the supplied measurements.

**Expected outputs:**

For available efficiency metrics:

```text
figures/flops_absolute.png
figures/memory_absolute.png
figures/cpu_latency_absolute.png
figures/cpu_throughput_absolute.png
```

If `--baseline-model` is specified, corresponding relative plots are also produced:

```text
figures/flops_relative.png
figures/memory_relative.png
figures/cpu_latency_relative.png
figures/cpu_throughput_relative.png
```

For available clinical metrics:

```text
figures/clinical_tp.png
figures/clinical_sensitivity.png
figures/clinical_balanced_accuracy.png
figures/clinical_youden_index.png
```

Only plots corresponding to columns present in the input CSVs are generated.

---

## Figure 4 — Few-shot learning curves

**Script:** `fig4.py`

**Purpose:** Plots few-shot AUROC curves for each dataset from a long-form results table.

**Required input schema:**

```text
dataset,model,shots,auc
```

- `dataset`: dataset or task name.
- `model`: model display name.
- `shots`: few-shot setting; may be numeric or a string such as `all`.
- `auc`: AUROC/AUC value in `[0, 1]`.

An optional `run` column may be included for repeated runs:

```text
dataset,model,shots,run,auc
```

When repeated runs are present, the script reports the arithmetic mean for each dataset/model/shot setting and computes the standard error across runs. Error bars can be disabled with `--no-error-bars`.

**Example command:**

```bash
python fig4.py --input fewshot_results.csv --output-dir figures
```

**Random seed:** None. No stochastic operations are used by the plotting pipeline.

**Expected outputs:**

```text
figures/fewshot_<dataset>.png
figures/fewshot_<dataset>.pdf
```

Dataset names are normalized for use in filenames.

---

## Extended Figures 1–3 — Confusion matrices

**Script:** `e-fig1_3.py`

**Purpose:** Loads confusion-matrix operating-point results, selects a common model-score threshold, and renders a grid of row-normalized confusion matrices.

**Required input schema:**

```text
threshold,task,model,regime,sensitivity,specificity,tp,fp,tn,fn
```

The script reads all CSV files in the directory supplied with `--data-dir`.

**Example command:**

```bash
python e-fig1_3.py --data-dir data --threshold 0.05 --output-dir figures
```

The requested threshold and threshold-matching tolerance can be configured with `--threshold` and `--threshold-tolerance`. Missing configured panels can be allowed with `--allow-missing`.

**Random seed:** None. Confusion-matrix values and derived metrics are computed deterministically from the supplied counts.

**Expected outputs:**

```text
figures/extended_figure1_3.png
figures/extended_figure1_3.pdf
figures/extended_figure1_3_summary.csv
```

The summary CSV records the selected threshold, count-derived FPR, sensitivity, specificity, balanced accuracy, Youden index, confusion-matrix counts, class totals, and source filename for each plotted panel.

---

## Tables

No standalone main-text table-generation script was identified among the provided files. If manuscript tables are generated programmatically, add one entry per table using the same structure:

```text
Table X
Script:
Required input schema:
Random seed:
Expected output artifact:
Example command:
```

If a table is assembled directly from one of the summary CSV files above, document that relationship explicitly here.

## Determinism

The plotting scripts documented above do not use stochastic data generation. Given identical input CSV files, package versions, and command-line arguments, they should produce the same numerical summaries. Minor rendering differences can occur across Matplotlib versions, operating systems, or font configurations.
