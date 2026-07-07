# Temporal Fusion Transformer Variants for Residential Energy Consumption Forecasting

An ablation study benchmarking four Temporal Fusion Transformer (TFT) variants against classical statistical models (ARIMA, SARIMA) and ML baselines (LSTM, DNN, ExtraTrees) for multi-horizon residential electricity demand forecasting.

## Overview

Residential energy consumption is highly chaotic compared to industrial load — it varies household to household based on individual usage patterns, unlike the steady, predictable demand seen in industrial datasets. This project explores whether the Temporal Fusion Transformer, with its attention-based architecture and native handling of multi-horizon, multi-covariate inputs, can outperform both traditional time-series methods and simpler deep learning baselines on this harder problem.

We built and compared **4 TFT variants** to isolate the effect of dimensionality reduction, hyperparameter search, and signal decomposition:

| Model | Feature Pipeline | Features | Params |
|-------|------------------|----------|--------|
| V1 — Baseline | 18 engineered features, no transform | 18 | 115.5k |
| V2 — PCA | V1 + 12 PCA components | 12 | 127k |
| V3 — PCA + Optuna | Same as V2, Optuna-tuned architecture | 12 | 89k |
| V4 — Wavelet + PCA | DWT sub-bands + PCA (12 PC) + cyclical time encoding | 42 | 178.6k |

## Dataset

- **Source**: Multi-meter residential consumption dataset (Uruguay), 15-minute resolution, Aug 2019 – Oct 2020
- **Size**: ~144,256 rows across multiple meter IDs after concatenation
- **Features per reading**: per-phase active/reactive power, phase voltage, current, power factor, cumulative energy
- **Preprocessing**: meter-wise forward/backward-fill for missing values, redundant phase columns dropped
- **Split**: chronological per-meter (70% train / up to 85% cumulative val / full series test) to prevent leakage

## Methodology

- **Architecture**: TFT with hidden size 32, 4 attention heads, dropout 0.1, hidden-continuous size 16, 7-quantile output via `QuantileLoss`, per-meter `GroupNormalizer`, 24-step (12hr) encoder window
- **V2**: 12 PCA components from 14 continuous base features (97.59% variance retained), fit on train split only
- **V3**: Optuna Bayesian search over learning rate, hidden size, attention heads, dropout, hidden-continuous size
- **V4**: Discrete Wavelet Transform (db4, level 3) sub-band extraction → PCA → cyclical (sin/cos) hour encoding to remove the hour 23→0 discontinuity

## Results

Test-set performance across all variants:

| Model | MAE (W) | RMSE (W) | R² | MAPE (%) |
|-------|---------|----------|-----|----------|
| TFT Baseline (V1) | 79.19 | 259.44 | 0.865 | 27.05 |
| TFT + PCA (V2) | 80.33 | 257.00 | 0.867 | 28.93 |
| TFT + PCA + Optuna (V3) | 84.43 | 261.28 | 0.863 | 27.76 |
| **TFT + Wavelet + PCA (V4)** | **78.57** | **257.02** | **0.867** | **23.58** |
| ARIMA (30-min) | 350.15 | 416.29 | −0.147 | 254.68 |
| ExtraTrees (w=28) | 282.78 | 370.00 | 0.330 | 160.40 |
| LSTM (w=16) | 297.50 | 385.69 | 0.272 | 163.90 |
| DNN (w=16) | 286.75 | 387.61 | 0.265 | 144.10 |

**Key findings:**
- V4 (Wavelet + PCA) is the best-performing variant, cutting MAPE by ~3.5 points over the baseline by capturing multi-scale load spikes that raw/PCA-only features miss.
- PCA alone (V2) slightly hurt MAE/MAPE — likely from dropping low-variance-but-useful features.
- Optuna hyperparameter search (V3) shrank the model (89k vs. 115.5k params) but didn't improve predictive quality, likely due to an overly broad search space.
- All TFT variants substantially outperform classical statistical baselines (ARIMA, SARIMA) and simpler ML/DL models, reducing MAE by up to 78% over ARIMA.

## Repository Structure

```
.
├── notebooks/           # Training notebooks for V1–V4
├── data/                 # Dataset processing scripts
├── report/               # Final paper (PDF)
└── README.md
```

## Team

| Name | Contribution |
|------|--------------|
| Rahul Saravanan | Architecture research, V1 baseline model |
| Aditi Pandey | V2 (PCA) and V3 (PCA + Optuna) models |
| Shishir Hegde | V3 wavelet integration, methodology & metrics compilation |
| Tejas Bajaj | V4 design & implementation (DWT feature extraction + PCA) |

## References

Base paper: S. Reddy et al., "Stacking deep learning and machine learning models for short-term energy consumption forecasting," *Advanced Engineering Informatics*, vol. 52, 2022.

TFT architecture: B. Lim et al., "Temporal fusion transformers for interpretable multi-horizon time series forecasting," *International Journal of Forecasting*, vol. 37, no. 4, 2021.

---
*Department of Information Technology, National Institute of Technology Karnataka, Surathkal*
