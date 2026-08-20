# 3 Peatland Probability Model

The peatland probability model is the foundational layer of the peatland characteristic mapping workflow. It predicts the likelihood that peat is present at every 10m pixel in Minnesota as a percentage from 0-1. The resulting probability raster was then thresholded into a binary peat mask, which was used as the masking boundary for all other property models, including peat depth, organic decomposition, and carbon stock.

## 3.1 Class Scheme

The model predicts a single binary target called `peat_binary`:

| Value | Meaning |
|---|---|
| 1 | Peat present (organic soil) |
| 0 | Peat absent (mineral soil) |

Training points are compiled from three primary sources (see 03-peatland-carbon-modeling ebook Ch. 2): the Minnesota DNR Peat Inventory, NRCS soil survey pedon data (NASIS), and The Minnesota Biological Survey (MBS). The compiled training set (`binary_peat_features_s1_combined.csv`) contains 57,134 points, with a class imbalance of about 3:1 negative:positive (non-peat:peat).

## 3.2 Algorithm & Training

### Algorithms evaluated

Three algorithms; Random Forest, XGBoost, and LightGBM were and the best performing model was selected as the final production model. Three non tree-based algorithms that were originally tested were dropped because they consistently underperformed the tree ensembles on spatial cross-validation AUC (GAM 0.954, SVM 0.953, Logistic Regression 0.945, CNN 0.89–0.96, versus RF/XGBoost/LightGBM all ≥0.956).

LightGBM was selected as the best performing algorithm for the probability model. The final
hyperparameters were:

- `n_estimators = 500`
- `learning_rate = 0.05`
- `num_leaves = 63`
- `subsample = 0.8`, `colsample_bytree = 0.8`
- Early stopping after 50 rounds without validation-fold improvement

### Sample design and class balancing

The ~3:1 negative:positive imbalance in the training data is addressed by
`scale_pos_weight`, set to the ratio of negative to positive training examples, which was the
same class-balancing approach used for the gradient boosting models in initial model runs.

### Feature selection pipeline

The covariate stack was reduced in two separate stages before model training:

1. **Correlation filter** 172 initial features were screened pairwise where Pearson |r| ≥ 0.90. The lower variance feature of the pair was dropped leaving 143 features.
2. **Recursive Feature Elimination (RFE)** was then run with iterative removal of the
   lowest importance feature evaluated against spatial CV performance. This further reduced the
   set of covariates to 37 features.

### Feature exclusions

Certain covariates were deliberately excluded from training regardless of the feature section.

- **Circular logic** layers that are peat classification products themselves (`histosols`, `npc_peatland_indicator`, gNATSGO organic-soil layers) were found in
  earlier experiments to inflate model results by creating a reference map rather than learning to predict the landscape. An example of this is an early random forest model run including these layers scored AUC = 0.994 which demonstrates circularity and not actuial predictive power
- **Polygon artifacts** one-hot encoded categorical layers (`quaternary_geology`,
  `pennockLandformClass`, `geomorphons`) resulted in geometric edge affects in spatial
  inference outputs that traced drawn polygon boundaries. Dropping these layers cost minimal AUC (~0.002 in the initial random forest comparison) for a substantial gain in visual performance.

