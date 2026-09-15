# 4 Peat Depth Model

The peat depth model predicts continuous peat thickness (cm) across the peatland
extent defined by the probability model (x03). It is the second stage of a
two-stage workflow: probability first establishes where peat is likely to occur,
and depth then estimates how thick the peat is within that masked area.

## 4.1 Target Variable and Training Data

The target, `depb`, is peat depth in centimeters. Depth is a masked regression problem where rather than training on the full point set, the model is trained only on locations that (1) have a measured depth greater than zero and (2) fall within the probability model's peat mask.

This two stage approach was implemted after finding that training the depth model on the
full point set (including 0 cm non-peat observations) produced falsely high R² values (~0.70) and predicted lower peat depths closer to the median. This result was misleading because the model was learning to separate 0 cm depths from non-zero depth points, not actually predicting the depth within peatlands. Restricting the input to training points that were already known to contain peat corrected the issue and gives a better measure of how well the model estimates depth.

**Masking rule:** `depb > 0` AND `PROB_LGBM_V2 ≥ 0.362`. This comes from the probability model's threshold selection (Ch. 3) which was used to define statewide peatland extent and also defines which points train the depth model.

**Sample size:** applying this filter to the combined training CSV resulted in 7,000 points used for depth model training. 

## 4.2 Algorithm and Training

### Algorithms evaluated

The same three tree-based algorithms compared for the probability model were
evaluated for depth: Random Forest, XGBoost, and LightGBM. Base (untuned) spatial and
random cross-validation results:

| Model | Spatial R² | Random R² | Spatial MAE | Random MAE |
|---|---|---|---|---|
| DEPTH_RF | 0.27 | 0.38 | ~76 cm | ~69 cm |
| DEPTH_XGB | 0.28 | 0.41 | ~75 cm | ~67 cm |
| DEPTH_LGBM | 0.29 | 0.43 | ~74 cm | ~66 cm |

LightGBM led on every metric both before and after hyperparameter tuning:

| Model | Spatial R² (tuned) |
|---|---|
| DEPTH_RF_TUNED | 0.2731 |
| DEPTH_XGB_TUNED | 0.2885 |
| **DEPTH_LGBM_TUNED (final)** | **0.2925** |

Tuning was performed with Optuna (TPE sampler, 50 trials per model, fixed seed),
optimizing for spatial CV R². Final production metrics for `DEPTH_LGBM_V2_TUNED`:

- **Spatial R²:** 0.2925
- **Random R²:** 0.4273
- **Spatial MAE:** 74.6 cm
- **Random MAE:** 66.1 cm

### Feature selection and retained/excluded covariates

Depth uses the same two-stage feature reduction as probability, a Pearson
correlation filter (|r| ≥ 0.90) followed by Recursive Feature Elimination (RFE) was applied to the remaining covariate stack. This reduced the original 143 features to 37 features after selection

Same as the probability workflow, RFE-selected features are used for the Random Forest model only. XGBoost and LightGBM are trained on the full correlation-filtered feature set because the boosting models handle correlated features internally.

The depth model shares the probability model's exclusion list (`quaternary_geology`,
`pennockLandformClass`, `geomorphons`, `gNATSGO`, `histosols`,
`npc_peatland_indicator`) for the same reasons (polygon artifacts and circular
logic) with one exception:

> `MN_organic_soils_classified_FIXED_snapped` is kept for depth modeling even though it was excluded
> for probability. Because this layer is a peat classification product it is 
> circular for the probability prediction (predicting if peat exists using a layer that
> says where peat is). But because the depth regression training is restricted 
> to the mask that was already identified as peat, using organic
> soil classification to predict how thick the peat is was beneficial to the depth models, and not circular.


## 4.3 Spatial Cross-Validation
The same 50 km block spatial cross-validation schema used for probability (Ch. 3.3)
is applied to depth. The gap between random and spatial R² is substantially larger for
depth (0.4273 vs. 0.2925) than it was for probability. This pattern is consistant across other model versions. In the earlier RF model comparisons LightGBM's spatial R² was 0.2991 against a random R² of 0.6206.

## 4.4 Outputs and Accuracy

### Depth surface

The statewide depth prediction was finalized as a Cloud-Optimized GeoTIFF:

- **Format:** `uint16`, 1:1 scale (pixel value = depth in cm)
- **Nodata:** 65535
- **Observed maximum:** 490 cm

All negative depth predictions are clipped to zero at inference (depth cannot be a negative number and gradient boosting/GAM/SVM regressors can result in small negative values near zero)

### Statewide depth distribution

| Statistic | Value |
|---|---|
| Mean | 138.3 cm |
| Median | 131.0 cm |
| Std. dev. | 44.9 cm |
| Maximum | 490.0 cm |
| % of peatland pixels > 100 cm | 79.3% |
| % of peatland pixels > 200 cm | 9.5% |
| % of peatland pixels > 300 cm | 0.4% |

| Depth class | Pixel count | % of peatland area |
|---|---|---|
| 0–50 cm | 343,579 | 0.1% |
| 50–100 cm | 59,894,884 | 19.6% |
| 100–200 cm | 215,652,211 | 70.5% |
| 200–300 cm | 28,552,931 | 9.3% |
| 300–500 cm | 1,248,987 | 0.4% |

The large majority of mapped peatland (70.5%) falls in the 100–200 cm depth class,
and depths beyond 300 cm are rare (0.4% of pixels) but present.

### Limitation: Deep peat

LightGBM regression predictions are capped at 366 cm, even though the model was trained on depth points that reached up to 900cm. This is a property of tree-ensemble regression where LightGBM is predicting the mean target value of the training points in each leaf, and the rarer deep peat observations get averaged together with shallower neighbor observations during training. Additionally, the tuned `min_child_samples = 90` constraint requires a larger minimum sample count per leaf and further promotes averaging. This means that the deepest peat deposits in Minnesota are likely being underpredicted by the statewide inference map.

### Downstream role

The depth surface feeds directly into the below-ground carbon stock calculation
(Ch. 6), where it is combined with bulk density and carbon percent to estimate carbon mass.

