# 4 Peat Depth Model

The peat depth model predicts continuous peat thickness (cm) across the peatland
extent defined by the probability model (x03). It is the second stage of a
two-stage workflow: probability first establishes where peat is likely to occur,
and depth then estimates how thick peat is within that masked area.

## 4.1 Target Variable and Training Data

The target, `depb`, is peat depth in centimeters. Depth is a masked regression problem where rather than training on the full point set, the model is trained only on locations that (1) have a measured depth greater than zero and (2) fall within the probability model's peat mask.

This two stage approach was established after finding that training the depth model on the
full point set (including 0 cm non-peat observations) produced deceptively high R² values (~0.70) and predicted lower peat depths closer to the median. This result was misleading because the model was learning to separate 0 cm depth from non-zero depth points, not actually predicting depth within peatlands. Restricting to training points already known to contain peat corrects this issue and gives a better measure of how well the model estimates peat depth.

**Masking rule:** `depb > 0` AND `PROB_LGBM_V2 ≥ 0.362`. This comes from the probability model's threshold selection (Ch. 3) which was used to define statewide peatland extent and also defines which points train the depth model.

**Sample size:** applying this filter to the combined training CSV resulted in ~7,000 points. 

## 4.2 Algorithm and Training

### Algorithms evaluated

The same three tree-based algorithms compared for the probability model were
evaluated for depth: Random Forest, XGBoost, and LightGBM. Base (untuned) spatial and
random cross-validation results:

| Model | Spatial R² | Random R² | Spatial MAE | Random MAE |
|---|---|---|---|---|
| DEPTH_RF_V2 | 0.27 | 0.38 | ~76 cm | ~69 cm |
| DEPTH_XGB_V2 | 0.28 | 0.41 | ~75 cm | ~67 cm |
| DEPTH_LGBM_V2 | 0.29 | 0.43 | ~74 cm | ~66 cm |

LightGBM led on every metric before hyperparameter tuning, and stayed as the winner after tuning:

| Model | Spatial R² (tuned) |
|---|---|
| DEPTH_RF_V2_TUNED | 0.2731 |
| DEPTH_XGB_V2_TUNED | 0.2885 |
| **DEPTH_LGBM_V2_TUNED (final)** | **0.2925** |

Tuning was performed with Optuna (TPE sampler, 50 trials per model, fixed seed),
optimizing directly for spatial CV R². Final production metrics for `DEPTH_LGBM_V2_TUNED`:

- **Spatial R²:** 0.2925
- **Random R²:** 0.4273
- **Spatial MAE:** 74.6 cm
- **Random MAE:** 66.1 cm

