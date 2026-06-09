# Machine Learning-Based Prediction of Photocatalytic Hydrogen Production

## Overview

This project applies Machine Learning techniques to predict photocatalytic hydrogen (H₂) production rates using catalyst composition, semiconductor properties, reaction conditions, and synthesis parameters collected from published literature.

The objective is to identify the most influential factors affecting hydrogen evolution and discover promising catalyst combinations for enhanced photocatalytic performance.

---

## Dataset

### Dataset Characteristics

- Total Experiments: 909
- Original Features: 28
- Final Features Used: 36
- Target Variable:
  - H₂ Production Rate (μmol g⁻¹ h⁻¹)

### Data Sources

The dataset was compiled from published photocatalytic hydrogen production studies containing:

- Cocatalyst information
- Semiconductor information
- Catalyst loading
- Bandgap energy
- Light source conditions
- Filter wavelength
- Solution volume
- Glycerol concentration
- Catalyst preparation parameters

---

## Machine Learning Pipeline

### Data Preprocessing

- Missing value handling
- Categorical encoding
- Target encoding
- Feature scaling where required

### Feature Engineering

Engineered features include:

- Power per volume
- Catalyst weight per load
- Bandgap × power interaction
- Catalyst load × glycerol interaction
- Log-transformed catalyst load
- Binary catalyst presence indicators

### Models Used

#### LightGBM

- 5-Fold Cross Validation
- Multi-seed ensemble training
- Regularization to reduce overfitting

#### CatBoost

- Native categorical feature handling
- 5-Fold Cross Validation
- Tuned depth and regularization

#### Final Ensemble

A simple weighted averaging ensemble:

```text
Final Prediction =
0.5 × LightGBM +
0.5 × CatBoost
```

---

## Final Model Performance

| Metric | Value |
|----------|----------|
| Train R² | 0.8742 |
| Test R² | 0.7187 |
| RMSE | 14,277.8 μmol g⁻¹ h⁻¹ |
| MAE | 5,098.1 μmol g⁻¹ h⁻¹ |
| Train-Test Gap | 0.1555 |

### Interpretation

The model explains approximately 72% of the variance in hydrogen production on unseen data while maintaining acceptable generalization performance.

---

## Feature Importance

Top factors influencing hydrogen production:

| Rank | Feature |
|--------|----------|
| 1 | Solution Volume |
| 2 | Preparation of Photocatalyst |
| 3 | Semiconductor Type |
| 4 | Cocatalyst Type |
| 5 | Photocatalyst Loading |
| 6 | Bandgap × Power |
| 7 | Power per Volume |
| 8 | Bandgap Energy |

These results indicate that both catalyst composition and reaction conditions significantly influence photocatalytic activity.

---

## Catalyst Analysis

### Best Observed Experimental Result

| Cocatalyst | Semiconductor | H₂ Production |
|------------|--------------|--------------|
| Pt | TiO₂ | 269,120 μmol g⁻¹ h⁻¹ |

### Top 5 Experimental Catalyst Systems

| Rank | Cocatalyst | Semiconductor | H₂ Rate |
|--------|------------|--------------|----------|
| 1 | Pt | TiO₂ | 269,120 |
| 2 | Pt | TiO₂ | 235,500 |
| 3 | NiSe₂ | TiO₂ | 219,200 |
| 4 | NiSe₂ | TiO₂ | 197,900 |
| 5 | La | ZnO | 184,800 |

---

## Best Catalyst Combinations (Average Performance)

| Rank | Combination | Mean H₂ Production |
|--------|------------|-------------------|
| 1 | NiSe₂/TiO₂ | 150,150 |
| 2 | Au-Pd/TiO₂ | 44,225 |
| 3 | Pd/TiO₂ | 41,320 |
| 4 | Cu/TiO₂ | 19,998 |
| 5 | Pt/TiO₂ | 17,462 |

---

## Model-Predicted Optimal Catalyst Systems

### No Filter

NiSe₂ + TiO₂

Predicted H₂ Production:

```text
20,418 μmol g⁻¹ h⁻¹
```

### UV-Vis (365 nm)

NiSe₂ + TiO₂

Predicted H₂ Production:

```text
18,862 μmol g⁻¹ h⁻¹
```

### Visible Light (420 nm)

NiSe₂ + ZnO

Predicted H₂ Production:

```text
5,663 μmol g⁻¹ h⁻¹
```

---


## Libraries Used

- Pandas
- NumPy
- Scikit-Learn
- LightGBM
- CatBoost
- Matplotlib

---

## Key Findings

- TiO₂ is the most effective semiconductor in the dataset.
- Pt/TiO₂ achieved the highest experimentally observed hydrogen production.
- NiSe₂/TiO₂ demonstrated the strongest average performance.
- Solution volume, catalyst preparation method, semiconductor type, and cocatalyst type are major determinants of hydrogen production.
- Machine learning can effectively accelerate photocatalyst screening and optimization.

---

## Future Work

- Expand dataset size with additional literature studies.
- Perform external validation on independent datasets.
- Develop explainable AI (SHAP) analysis.
- Experimentally validate model-predicted catalyst systems.
- Deploy the trained model as a web application.

---

## Author

Palak Sibbal

Machine Learning for Photocatalytic Hydrogen Production Research
