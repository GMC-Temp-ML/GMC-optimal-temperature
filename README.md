# GMC-optimal-temperature

Reproducibility package for sequence-based prediction of the optimal temperature of glucose oxidase (GOX). Features are ESM-2 (`esm2_t30_150M_UR50D`) mean-pooled embeddings and amino-acid composition (AAC); the regressor is LightGBM; evaluation is leave-one-out cross-validation (LOOCV).

## Summary of results

> **The predictive gain is small and not statistically established.**

LOOCV on 46 usable GOX sequences:

| Model | RMSE (°C) |
|---|---|
| Mean baseline | 13.64 |
| AAC | 13.74 |
| ESM-2 | 13.18 |
| AAC + ESM-2 | 12.95 |

For AAC + ESM-2: R² = 0.058, Spearman ρ = 0.300, permutation test p = 0.079. The improvement over the baseline is about 0.7 °C. These values do not support a claim of an accurate predictor.

## Repository contents

| File | Description |
|---|---|
| `project_1_clean.ipynb` | Analysis notebook (46 cells). Erroneous output in cell 9 (CD-HIT) was cleared; code is unchanged. |
| `opt_train_test.csv` | General optimal-temperature dataset: 1896 rows, columns `seq` and `label`. Input to notebook cell 1. |
| `56_GOX.csv` | 56 GOX entries (UniProt IDs). Extra columns are dropped in cell 2; the model uses sequence and optimal temperature only. |
| `all_sequences_clean.fasta` | Sequences used for CD-HIT clustering (expected: 1932 records). |
| `clustered_output.clstr` | CD-HIT cluster file: 1932 sequence IDs, 1739 clusters (identity threshold 0.7). Two outlier sequences were excluded. |
| `general_gox_aac_esm.csv` | Feature table (1930 rows; 40 duplicate AAC columns and 640 ESM-2 columns). |
| `gox_final_results_table.csv` | Final LOOCV results. |
| `gox_results_summary.json` | Summary of final results. |
| `requirements.txt` | Package versions of the local modeling environment (Colab versions as comments). |
| `LICENSE` | MIT License (code). |

## Computational environment

The pipeline was executed in two environments. Versions below are as reported by the authors and are not independently re-verified.

| Stage | Environment | Versions |
|---|---|---|
| CD-HIT clustering (cell 9) | Google Colab | CD-HIT 4.8.1 (checked on Colab; not confirmed for the original run) |
| ESM-2 embeddings (cells 16–18) | Google Colab, NVIDIA T4 GPU | Python 3.13, torch 2.14.1+cu130, fair-esm 2.0.0 |
| Modeling (cell 19 onward) | Local, Windows, VS Code | Python 3.8.8, torch 2.4.1+cpu, fair-esm 2.0.0, lightgbm 4.6.0, scikit-learn 1.3.2, optuna 4.5.0, pandas 2.0.3, numpy 1.24.4, scipy 1.10.1 |

CD-HIT command used in cell 9:

```bash
cd-hit -i all_sequences_clean.fasta -o clustered_output -c 0.7 -n 5 -d 0
```

## Method

- **Dataset:** 56 GOX UniProt entries; 46 usable after filtering.
- **Sequence length:** sequences longer than 1022 residues are truncated to fit the ESM-2 context. This affects 56 of 1930 sequences (2.9%) in the general GOX-set table `general_gox_aac_esm.csv`, and 1 of the 56 GOX entries in `56_GOX.csv` (maximum length 1141 aa).
- **Features:** 20-dimensional AAC and 640-dimensional mean-pooled ESM-2 embeddings, reduced by PCA.
- **Model:** LightGBM regressor; hyperparameters tuned with Optuna (TPE sampler, 50 trials).
- **Seeds:** 42 and 0–9.
- **Validation:** LOOCV. Sequences from the same CD-HIT cluster can appear in both training and test folds: 18 of 46 sequences fall into 8 shared clusters. Results should be interpreted with this in mind.
- **Oracle reference:** on the 18-sequence subset, an oracle-style reference reaches RMSE 12.67 °C (notebook cell 23).

## Data sources and licenses

| Data | Source | License |
|---|---|---|
| `opt_train_test.csv` | Public Hugging Face dataset `biomap-research/optimal_temperature` (identical copy) | Apache-2.0 |
| `56_GOX.csv`, `gox_*` tables, derived features | Compiled by the authors from UniProt and BRENDA (as reported by the authors) | UniProt: CC BY 4.0 ([uniprot.org/help/license](https://www.uniprot.org/help/license)). BRENDA: CC BY 4.0 ([brenda-enzymes.org/license.php](https://www.brenda-enzymes.org/license.php)). Redistribution is permitted with attribution; please cite UniProt and BRENDA when reusing these data. |
| Code (`.ipynb`) | This repository | MIT ([LICENSE](LICENSE)) |

Physicochemical parameters in `56_GOX.csv` (pH optimum, Km, kcat, temperature range) originate from BRENDA and UniProt. Whether each value is measured or imputed has not yet been confirmed, and these parameters are not used by the final model.

## How to reproduce

1. Install the local modeling environment from `requirements.txt`, or use the Colab environment listed above for stages 1–2.
2. Place `opt_train_test.csv` in the same directory as the notebook.
3. **Modeling only (fastest):** start at cell 19 with `general_gox_aac_esm.csv`.
4. **Full pipeline:** run cells 9–18 (CD-HIT and ESM-2 embeddings) with `all_sequences_clean.fasta`, then cell 19 onward.

Note: LightGBM failed to load in a sandbox environment (`libgomp` / `lib_lightgbm.so` error). Reproduction on a standard Python installation with the listed versions has not been tested in this repository.

## Known limitations

- Small sample size (n = 46); LOOCV estimates have high variance.
- Cluster overlap between training and test folds (see Method).
- The improvement over the mean baseline is not statistically significant.
- Stages were run in different environments; results have not been reproduced independently.
- The measured-versus-imputed status of the physicochemical parameters in `56_GOX.csv` (pH, Km, kcat, temperature range) is not yet confirmed.

## Citation

A citation entry and Zenodo DOI will be added with the v1.0.0 release.
