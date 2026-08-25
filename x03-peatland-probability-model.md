# 3 Peatland Probability Model

The peatland probability model is the foundational layer of the peatland characteristic mapping workflow. It predicts the likelihood that peat is present at every 10m pixel in Minnesota as a percentage from 0-1. The resulting probability raster was then thresholded into a binary peat mask, which was used as the masking boundary for all other property models, including peat depth, organic decomposition, and carbon stock.

## 3.1 Class Scheme

The model predicts a binary target called `peat_binary`:

| Value | Meaning |
|---|---|
| 1 | Peat present (organic soil) |
| 0 | Peat absent (mineral soil) |

Training points are compiled from three primary sources (see 03-peatland-carbon-modeling ebook Ch. 2): the Minnesota DNR Peat Inventory, NRCS soil survey pedon data (NASIS), and The Minnesota Biological Survey (MBS). The compiled training set (`binary_peat_features_s1_combined.csv`) contains 57,134 points, with a class imbalance of about 3:1 negative:positive (non-peat:peat).

## 3.2 Algorithm & Training

### Algorithms

Three different algorithms; Random Forest, XGBoost, and LightGBM were run to predict peat probability and the best performing model was selected as the final production model. Three non tree based algorithms that were originally tested were dropped because they consistently underperformed on spatial cross-validation AUC (GAM 0.954, SVM 0.953, Logistic Regression 0.945, CNN 0.89–0.96, versus RF/XGBoost/LightGBM all ≥0.956) or had issues during spatial inference creation.

LightGBM was selected as the best performing algorithm for the probability model. The final
hyperparameters were:

- `n_estimators = 500`
- `learning_rate = 0.05`
- `num_leaves = 63`
- `subsample = 0.8`, `colsample_bytree = 0.8`


### Sample design and class balancing

The ~3:1 negative:positive imbalance in the training data is addressed by
`scale_pos_weight`, set to the ratio of negative to positive training examples, which was the
same class-balancing approach used for the gradient boosting models in initial model runs.

### Feature selection pipeline

The covariate stack was reduced in two separate stages before model training:

1. **Correlation filter** 172 initial features were compared in pairs where Pearson |r| ≥ 0.90. The lower variance feature of the pair was dropped leaving 143 features.
2. **Recursive Feature Elimination (RFE)** was then run with iterative removal of the
   lowest importance feature evaluated against spatial cross validation. This further reduced the
   set of covariates to 37 total features.

### Feature exclusions

Certain covariates were excluded from training regardless of the feature section outcomes.

- **Circular logic** Layers that are peat classification products themselves (`histosols`, `npc_peatland_indicator`, gNATSGO organic-soil layers) were found to inflate model results by creating a reference map rather than learning to predict the landscape. An example of this is an early random forest model run that included these layers scored AUC = 0.994 which demonstrates circularity and not actual predictive power.
- **Polygon artifacts** one-hot encoded categorical layers (`quaternary_geology`,
  `pennockLandformClass`, `geomorphons`) resulted in edge affects in the spatial
  inference outputs that traced drawn polygon boundaries. Dropping these layers cost minimal AUC (~0.002 in the initial random forest comparison) for a noticable gain in visual performance.


## 3.3 Spatial Cross-Validation

Two cross-validation methods were run for every model in this project and
both are reported.

**Random cross-validation** (stratified 5-fold KFold, shuffled) was used as the conventional validation approach. However, because random grouping can place geographically close observations in both training and validation folds, the model can use spatially correlated observations which may cause interpolation to happen resulting in artificially high performance estimates.**run on sentence**



**Spatial block cross-validation** is used as the primary decision making metric.
The state of Minnesota was divided into a 50 km × 50 km grid with all points in the same grid block being placed in the same fold. Blocks are then distributed across 5 folds. 
```
block_x = floor(easting / 50000)
block_y = floor(northing / 50000)
block_id = block_x * 10000 + block_y
# unique block_ids shuffled (fixed seed) and assigned round-robin to 5 folds
```

Because an entire 50 km block is withheld at once, the model cannot always use nearby
training points during validation. This produces a lower but more accurate estimate of mapping. 

The difference between random and spatial CV scores also helps to quantify the degree of spatial autocorrelation that exists (For reference, the gap is substantially larger for peat depth than for peat presence, ~0.29 spatial vs. ~0.43 random R² for the depth model)


## 3.4 Outputs & Accuracy

### Probability surface

The model is then used to create a spatial inference which is a continuous statewide probability of peat raster (10 m, EPSG:5070). This raster was then encoded as a Cloud Optimized GeoTIFF (COG) to create a smaller file for distribution and access. 

- **Format:** `uint8`, where `pixel value × 0.005 = probability (0–1)`
- **Nodata:** 255

### Threshold selection

Because downstream models and acreage require a binary peat/non peat surface,
the continuous probability output was thresholded. The threshold was chosen by
sweeping 500 cutoffs (0.01–0.99) across spatial-CV out of fold predictions
and evaluating each against several other metrics: 

**F1**
**Matthews Correlation Coefficient (MCC)**
**Youden's J**
**Accuracy**
**Balanced Accuracy**

The final threshold selected for the probability model is the median across these
metrics: 0.362 (the mean of the MCC/Youden/Accuracy-optimal thresholds was 0.364).
This differs from the single-metric (F1-optimal) threshold approach that was used in
the initial random forest pipeline which was 0.327. Using multiple metrics rather than just the F1 score was chosen to prevent selecting a threshold that looks good on one criteria but is poorly balanced on others.

For reference, the other probability algorithms had their own F1-optimal thresholds:

| Model | Threshold |
|---|---|
| PROB_RF_V2 | 0.452 |
| PROB_XGB_V2 | 0.450 |
| **PROB_LGBM_V2 (final)** | **0.362** |

### Accuracy

- **Spatial CV AUC:** ~0.97
- **Brier score:** 0.0383

**add full confusion matrix, per-class accuracy, precision/recall/F1 at the
0.362 threshold, and variable importance ranking**

### Mapped extent and threshold sensitivity

Applying the 0.362 threshold statewide results in a mapped peatland extent of:

| Metric | Value |
|---|---|
| Acres | 7,846,842 |
| Hectares | 3,175,507 |
| Square kilometers | 31,755 |

The mapped extent is dependent on the threshold that is used, which is why multiple metrics were used in the final approach.

| Probability threshold | Mapped acres |
|---|---|
| ≥ 0.25 | 11,340,267 |
| ≥ 0.33 | 8,469,370 |
| **≥ 0.362 (final)** | **7,846,842** |
| ≥ 0.45 | 6,793,497 |
| ≥ 0.50 | 6,335,711 |

### Downstream role

The thresholded peat mask (probability ≥ 0.362) defines the spatial mask for every
other property model that was created by this project: peat depth, organic decomposition
(Fibric/Hemic/Sapric/Mineral percent), and carbon stock estimate.


