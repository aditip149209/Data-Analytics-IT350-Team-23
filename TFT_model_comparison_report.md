# TFT Models and Baseline Methods: Comparative Results Report

## 1. Scope and Sources

This report summarizes results from:

- `TFT Notebooks` (our method variants).
- `Experiments` (older baseline methods reproduced from base-paper style notebooks).

Primary metric artifacts used:

- `TFT Notebooks/Non-Wavelet Version/tft_no_wavelets_metrics.csv`
- `TFT Notebooks/Wavelet PCA Version/tft_wavelet_pca_metrics.csv`
- `TFT Notebooks/Wavelet PCA Version/tft_wavelet_pca_run_info.csv`
- Saved notebook outputs in:
  - `TFT Notebooks/Non Wavelet WIth PCA/nonWaveletWithPCA.ipynb`
  - `Experiments/gb-base-paper.ipynb`
  - `Experiments/xgb-base-paper.ipynb`
  - `Experiments/sarima-base-paper.ipynb`
- `Experiments/paper_arima_metrics.csv`

Note on ARIMA baseline: final ARIMA values are now taken from `Experiments/paper_arima_metrics.csv`.

## 2. Our TFT Variants (V1 to V4)

I map your implementation evolution as follows:

- V1: TFT without wavelets, without PCA
  - `TFT Notebooks/Non-Wavelet Version/tft-notebook-non-wavelet-design.ipynb`
- V2: TFT without wavelets, with PCA
  - `TFT Notebooks/Non Wavelet WIth PCA/nonWaveletWithPCA.ipynb`
- V3: TFT without wavelets, with PCA + Optuna tuning
  - `TFT Notebooks/Non Wavelet WIth PCA/nonWaveletWithPCAwithOptuna.ipynb`
- V4: TFT with wavelet features + PCA (final rich-feature variant)
  - `TFT Notebooks/Wavelet PCA Version/wavelet-tft-with-pca.ipynb`

### 2.1 Metrics Table (Validation/Test)

| Version | Variant                        | Split           |           MAE |          RMSE |            R2 |       MAPE(%) |
| ------- | ------------------------------ | --------------- | ------------: | ------------: | ------------: | ------------: |
| V1      | Non-wavelet TFT                | Validation      |     82.010757 |    264.254893 |      0.868015 |     26.444376 |
| V1      | Non-wavelet TFT                | Test            |     79.186516 |    259.443711 |      0.864926 |     27.047752 |
| V2      | Non-wavelet TFT + PCA          | Validation      |     83.196289 |    261.641080 |      0.870613 |     28.251335 |
| V2      | Non-wavelet TFT + PCA          | Test            |     80.330658 |    257.003952 |      0.867455 |     28.925804 |
| V3      | Non-wavelet TFT + PCA + Optuna | Validation/Test | Not persisted | Not persisted | Not persisted | Not persisted |
| V4      | Wavelet + PCA TFT              | Validation      |     81.728790 |    261.677536 |      0.870577 |     23.667265 |
| V4      | Wavelet + PCA TFT              | Test            |     78.569626 |    257.021962 |      0.867436 |     23.576200 |

### 2.2 V1 -> V4 Evolution Analysis

#### V1 -> V2 (add PCA on engineered continuous features)

Observed behavior on Test:

- RMSE improved: 259.44 -> 257.00 (better).
- R2 improved: 0.8649 -> 0.8675 (better).
- MAE worsened: 79.19 -> 80.33.
- MAPE worsened: 27.05 -> 28.93.

Interpretation:

- PCA helped overall variance fit (RMSE/R2), but slightly hurt pointwise absolute/relative error behavior.
- This usually indicates better smoothing/global structure capture but less sharp local accuracy.

#### V2 -> V3 (add Optuna tuning)

- The notebook contains Optuna objective/trials and retraining logic.
- However, final `metrics_df` output values are not persisted in current notebook outputs.
- So V3 is an implementation step in the pipeline, but not a reportable numeric checkpoint from available artifacts.

#### V2/V3 -> V4 (introduce wavelet descriptors + PCA)

On Test, compared to V2:

- MAE improved: 80.33 -> 78.57.
- RMSE stayed essentially tied: 257.00 -> 257.02.
- R2 stayed essentially tied: 0.8675 -> 0.8674.
- MAPE improved strongly: 28.93 -> 23.58.

Interpretation:

- Wavelet features provided a large gain in relative error stability (MAPE), likely by improving multi-scale signal representation.
- Absolute and variance metrics remained competitive/tied with best PCA-only run.

#### End-to-end V1 -> V4 summary (Test)

- MAE: 79.19 -> 78.57 (improved).
- RMSE: 259.44 -> 257.02 (improved).
- R2: 0.8649 -> 0.8674 (improved).
- MAPE: 27.05 -> 23.58 (major improvement).

The main practical gain is MAPE reduction while preserving RMSE/R2 gains.

## 3. Baseline Methods (Experiments Folder) vs Our Methods

Baselines inspected:

- `Experiments/gb-base-paper.ipynb`
- `Experiments/xgb-base-paper.ipynb`
- `Experiments/sarima-base-paper.ipynb`
- `Experiments/arima-base-paper.ipynb`

### 3.1 Extracted Baseline Results

#### GB stacking notebook (`gb-base-paper.ipynb`)

Best parsed row from persisted output:

- Model: ExtraTrees (as candidate in stacked setup)
- Window: 28
- MAE: 282.775557
- RMSE: 370.003100
- R2: 0.329888
- MAPE(%): 160.377532

#### XGB stacking notebook (`xgb-base-paper.ipynb`)

Best parsed row from persisted output:

- Model: ExtraTrees
- Window: 28
- MAE: 282.775557
- RMSE: 370.003100
- R2: 0.329888
- MAPE(%): 160.377532

#### SARIMA/ETS notebook (`sarima-base-paper.ipynb`)

Persisted final model table includes:

- SARIMA: MAE 66836.1568, RMSE 77284.2127, R2 -0.2643, MAPE 140.2579
- ETS: MAE 368311.4292, RMSE 378754.6976, R2 -29.3655, MAPE 665.1886

#### ARIMA notebook (`arima-base-paper.ipynb`)

- Final exported metrics (`paper_arima_metrics.csv`):
  - Model: ARIMA
  - BestOrder: (5, 0, 4)
  - MAE: 69593.2508
  - RMSE: 76799.6922
  - R2: -0.2485
  - MAPE(%): 138.5390

### 3.2 Related-work style comparison (baselines vs our final variant)

Using best baseline row available from GB/XGB outputs (RMSE 370.00, MAE 282.78, R2 0.3299, MAPE 160.38) against V4 Test (RMSE 257.02, MAE 78.57, R2 0.8674, MAPE 23.58):

- RMSE reduction: about 30.5%.
- MAE reduction: about 72.2%.
- MAPE reduction: about 85.3%.
- R2 increase: +0.5375 absolute.

Takeaway:

- Your TFT family is dramatically better than the reproduced older baseline notebooks on all persisted comparable metrics.
- V4 is strongest as an overall compromise: best relative error regime while keeping top RMSE/R2 quality.
- ARIMA and SARIMA are in a very similar error range in these artifacts, and both are far behind the TFT variants.

## 4. Base Dataset Used in Our Method

Across TFT notebooks, the common base dataset is:

- `total_consumption_data.csv`

Typical path usage:

- Local relative path: `../../total_consumption_data.csv` in non-wavelet notebooks.
- Kaggle-attached paths with fallback search in the wavelet notebook.

Wavelet-PCA run info confirms scale used in that run:

- Train samples: 100,977
- Validation samples: 122,615
- Test samples: 144,256
- Feature count: 42
- Wavelet type: db4
- Wavelet decomposition level: 3
- PCA components: 12

## 5. Common Preprocessing and Data Quality Improvements in Our TFT Pipeline

The following steps are consistently present (with small variant-specific additions):

1. Column cleanup and typing

- Strip column names.
- Parse `datetime` robustly (UNIX timestamp or already formatted string).
- Coerce non-ID columns to numeric.

2. Temporal ordering and meter-wise consistency

- Sort by `meter_id`, then `datetime`.
- Fill missing numeric values per meter using forward-fill and backward-fill.

3. Feature pruning / sanitization

- Drop irrelevant or redundant phase columns depending on variant.
- Replace `inf/-inf` with NaN.
- Fill NaNs in model feature columns (median in PCA variants).
- Remove rows with missing critical identifiers (`datetime`, `meter_id`).

4. Stable entity encoding

- Create compact categorical meter label (`M000`, `M001`, ...).

5. Leakage-safe chronological indexing and splits

- Build `time_idx` by cumulative count per meter.
- Compute per-meter cutoffs:
  - train cutoff = 70%
  - validation cutoff = 85%
- Keep chronological integrity in splits.

6. Sequence eligibility filtering

- Ensure each meter has enough history for encoder + prediction windows.

7. TFT dataset design

- Known reals include temporal/calendar features (`time_idx`, hour/day flags).
- Unknown reals include target and dynamic sensor features.
- Group normalization by `meter_id` with `GroupNormalizer`.

8. Training controls for robustness

- Early stopping on validation loss.
- Gradient clipping.
- Fixed random seeds for repeatability.

## 6. What Changes Across Variants (Implementation Evolution)

### V1 (non-wavelet, no PCA)

- Core TFT with engineered electrical + temporal features.

### V2 (non-wavelet + PCA)

- Adds `StandardScaler` + PCA on continuous features.
- PCA fit only on training partition to avoid leakage.
- Appends principal components into TFT feature set.

### V3 (non-wavelet + PCA + Optuna)

- Adds Bayesian hyperparameter search (Optuna) for TFT config.
- Trains final TFT using best trial parameters.
- Metrics not persisted in current artifact state.

### V4 (wavelet + PCA)

- Adds wavelet decomposition-derived descriptors (approximation/detail-energy/detail-std) for selected power/energy signals.
- Keeps leakage-safe PCA stage after wavelet/base feature construction.
- Produces best relative-error profile (MAPE) among persisted runs.

## 7. Recommended Wording Direction for Your LaTeX Sections

If you want to mirror this in `latex.tex`, a clear narrative is:

1. Start from V1 as the plain TFT baseline.
2. Explain why PCA (V2) was introduced and what metrics improved/worsened.
3. Explain Optuna step (V3) as optimization infrastructure (state clearly that persisted metrics were unavailable in this snapshot).
4. Present V4 as final representation-rich model with wavelet + PCA.
5. In related work comparison, emphasize older baselines show much weaker fit (high RMSE/MAE/MAPE, low R2) than TFT variants on persisted results.

## 8. Reproducibility Notes and Gaps

- Metrics for V1, V2, V4 are available and stable from saved outputs/CSVs.
- V3 Optuna final metrics are currently missing from saved outputs.
- ARIMA baseline metrics are available via `Experiments/paper_arima_metrics.csv` (BestOrder (5, 0, 4)).

If needed, the remaining main gap can be closed by rerunning the Optuna notebook and exporting fixed final metrics.
