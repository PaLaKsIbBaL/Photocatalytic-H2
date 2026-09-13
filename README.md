# Photocatalytic H₂ Production Rate Prediction

A stacked ensemble (CatBoost + Random Forest + XGBoost) that predicts photocatalytic hydrogen production rate from reaction and catalyst design variables, trained on 909 experiments aggregated from 119 literature sources. Includes SHAP-based interpretability to identify the dominant physical/chemical drivers of H₂ yield.

**R² = 0.776 | MSE reduced 32% vs. best single baseline model**

---

## Table of Contents
- [Overview](#overview)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Results](#results)
- [SHAP Feature Importance](#shap-feature-importance)
- [Repository Structure](#repository-structure)
- [Installation & Usage](#installation--usage)
- [Limitations](#limitations)
- [References](#references)
- [License](#license)

---

## Overview

Photocatalytic hydrogen evolution is a promising route to clean H₂ production, but identifying effective catalyst/reaction-condition combinations experimentally is slow and expensive. This project builds a machine learning surrogate model to predict H₂ production rate (μmol g⁻¹ h⁻¹) from catalyst composition, preparation method, and reaction conditions, aiming to help narrow the experimental search space.

Three tree-based regressors — **CatBoost**, **Random Forest**, and **XGBoost** — are trained individually and combined via **out-of-fold (OOF) stacking with a Ridge meta-model**. Model behavior is interpreted using **SHAP (SHapley Additive exPlanations)**.

## Dataset

- **909 experiments** aggregated from **119 distinct literature references**
- Target: H₂ production rate, range **0–269,120 μmol g⁻¹ h⁻¹** (strongly right-skewed; mean 11,747, median 1,711)
- **26 raw variables** → 11 categorical (catalyst type, cocatalyst, light source, preparation method, etc.) + 15 numeric (loading %, pH, calcination temp/time, bandgap, reaction temp, etc.)
- **68 total features** after preprocessing: 11 categorical + 15 numeric + 15 missing-value indicator flags + 27 engineered interaction/ratio/log features

> Raw data is not redistributed in this repository due to source licensing. See [Installation & Usage](#installation--usage) for how to point the script at your own copy.

## Methodology

| Component | Approach |
|---|---|
| Target transform | log(1+y) to handle heavy right-skew; predictions back-transformed before scoring |
| Categorical handling | CatBoost: native ordered target statistics. RF/XGBoost: leakage-safe K-fold target encoding |
| Missing data | Per-field missing-value indicator flags + train-only median imputation |
| Feature engineering | 27 domain-informed features (e.g. power-per-volume, load × glycerol interaction, log-transformed loading/bandgap) |
| Validation | 80/20 hold-out split (727 train / 182 test) + 5-fold CV for stacking |
| Ensembling | Ridge meta-model (non-negative coefficients) fit on out-of-fold base-model predictions |
| Interpretability | SHAP TreeExplainer on the CatBoost model |

**Hyperparameters**

| Model | Key settings |
|---|---|
| CatBoost | depth=9, learning_rate=0.025, l2_leaf_reg=4, grow_policy=Lossguide, max_leaves=200 |
| Random Forest | n_estimators=500, max_features=sqrt, min_samples_leaf=2 |
| XGBoost | n_estimators=2000 (early-stopped), learning_rate=0.03, max_depth=6 |
| Ridge meta-model | alpha=1.0, non-negative coefficients |

## Results

| Model | R² | MSE | RMSE | MAE |
|---|---|---|---|---|
| CatBoost | 0.7573 | 161,613,695 | 12,712.7 | 4,524.8 |
| Random Forest | 0.6023 | 264,854,457 | 16,274.3 | 5,546.8 |
| XGBoost | 0.6514 | 232,151,701 | 15,236.5 | 5,306.7 |
| Simple average | 0.6707 | 219,272,109 | 14,807.8 | 5,032.0 |
| **Stacked ensemble (Ridge)** | **0.7762** | **149,038,326** | **12,208.1** | **4,337.0** |

The stacked ensemble outperformed every individual model. The Ridge meta-model learned weights of **0.871 (CatBoost), 0.000 (Random Forest), 0.164 (XGBoost)** — CatBoost's native categorical handling made it the dominant contributor.

![Model comparison](figures/model_comparison.png)
![Parity plot](figures/parity_plot.png)

## SHAP Feature Importance

The `Reference` field (source publication) was the single most influential feature — expected, since experiments from the same paper share reactor design and measurement protocol. Beyond that, catalyst composition variables (`Semiconductor 1/2`, `Cocatalyst 1`, `Structure`, preparation method) dominate the ranking, consistent with physical intuition that catalyst identity and synthesis route are primary drivers of photocatalytic activity.

![SHAP summary](figures/shap_summary_beeswarm.png)
![SHAP bar chart](figures/shap_summary_bar.png)
![SHAP dependence plots](figures/shap_dependence_grid.png)

## Repository Structure

```
.
├── README.md
├── requirements.txt
├── LICENSE
├── src/
│   └── h2_model.py          # full training + stacking + SHAP pipeline
├── figures/
│   ├── shap_summary_beeswarm.png
│   ├── shap_summary_bar.png
│   ├── shap_dependence_grid.png
│   ├── parity_plot.png
│   └── model_comparison.png
└── docs/
    └── methodology.md        # full written methodology & discussion
```

## Installation & Usage

```bash
git clone https://github.com/<your-username>/h2-photocatalytic-prediction.git
cd h2-photocatalytic-prediction
pip install -r requirements.txt
```

Update the `DATA_PATH` variable in `src/h2_model.py` to point at your dataset, then run:

```bash
python src/h2_model.py
```

This trains all three base models, generates out-of-fold stacking predictions, fits the Ridge meta-model, evaluates on the held-out test set, and saves SHAP/parity/comparison plots.

## Limitations

- **Provenance leakage**: experiments from the same source paper can appear in both train and test splits, so the `Reference` feature's importance partly reflects paper-level memorization rather than pure mechanistic signal.
- **Dataset heterogeneity**: aggregating 119 sources with differing reactors, units, and protocols inflates absolute error metrics (MSE, MAE) even where relative fit (R²) is reasonable.
- Random Forest contributed negligible unique signal to the final ensemble (meta-model weight = 0.000); a two-model CatBoost + XGBoost ensemble would likely perform comparably at lower compute cost.

## References

1. Bakır, R., Orak, C., Yüksel, A. (2024). Optimizing hydrogen evolution prediction: A unified approach using random forests, lightGBM, and Bagging Regressor ensemble model. *International Journal of Hydrogen Energy*, 67, 101–110. https://doi.org/10.1016/j.ijhydene.2024.04.173
2. Suriyaprakash, J., Saudagar, A.K.J., Wu, L., Shan, L. (2025). Machine learning-powered nanoengineering of flexible, durable and scalable photoelectrode for efficient H₂ production. *International Journal of Hydrogen Energy*, 189, 152214. https://doi.org/10.1016/j.ijhydene.2025.152214

## License

MIT — see [LICENSE](LICENSE) for details.
