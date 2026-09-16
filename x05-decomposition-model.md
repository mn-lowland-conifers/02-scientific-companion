# 5 Organic Decomposition Model

The organic decomposition model predicts the composition of peat in the top
meter of the soil profile as a percentage of four classes: fibric, hemic, sapric, and mineral. The probability model answers "is there peat here?" and the depth model answers "how deep is it?." The composition model gets at the question "how decomposed is the material?" This distinction matters because more decomposed sapric peat is often denser and more carbon concentrated than the lesser decomposed fibric peat.

## 5.1 Target Variable and Training Data

Four targets are modeled: `Fibric_pct`, `Hemic_pct`, `Sapric_pct`, and `Mineral_pct`. The 'Mineral_pct' variable takes into account the mineral fraction of profiles that are shallower than 100cm. 
Training data comes from the MN DNR Peat Inventory with horizon level decomposition classification, and then restricted to the top meter of the profile. The `Unknown_pct ', which is an uncertain category in the raw peat inventory data was dropped, and the four classes were normalized to sum to 100% per pedon.

**Sample size:** the final training set data (`organic_composition_features_combined.csv`)
contains 18,041 points.

**Masking:** same as the workflow in the depth modeling, the composition training and spatial inference are restricted to the
probability model peat mask, using the same threshold of 0.362 (Ch. 3–4).

## 5.2 Modeling Approach: Predicting a Compositional Target

Predicting multiple percentages that must sum to 100% is a compositional problem,
and the two different strategies were tested before implementing the final
approach.

**Independent target modeling** (03-peatland-carbon-modeling, `exp421`/`exp422`) trains a
separate regressor for each of the classes with no constraint that the output of all targets sums to 100%. This approach lets each regressor optimize independently, but because each process is independent from one another the predictions across the classes will not sum to 100%. 

**Iterative residual regression** (03-peatland-carbon-modeling, `exp426`/`COMP_RF_001`) predicts Fibric % first, then Hemic % as a fraction of the remainder (100% − Fibric), and finally assigns Sapric % whatever is left over. This means that the outputs will sum exactly to 100%, but adds an order dependency. Errors in the first predicted class (Fibric) continue into each class predicted after it, and the last class to be predicted (Sapric in this example) absorbs the combined error of everything predicted before it rather than being independently modeled.

In the initial RF model random-CV comparison, independent modeling
outperformed iterative residual regression (R² = 0.4341 vs. 0.3999)

**Selected approach:** The pipeline chosen uses independent per-target regression for all four classes, and then applies a post-inference normalization step to constrain to a sum of 100% after the prediction instead of during it. This method utilizes the accuracy advantage of independent modeling with the guarantee sum of iterative regression, while avoiding the order dependency problem. No one class has an advantage over another, and expanding from three classes to four (adding Mineral) required no change to the modeling logic.

## 5.3 Algorithm and Training

The same three algorithms compared for probability and depth were compared for each
composition target. Target specific feature sets from RFE were applied only to the Random Forest models.

| Target | RFE features | RFE spatial R² |
|---|---|---|
| Fibric | 24 | 0.1053 |
| Hemic | 30 | 0.0783 |
| Sapric | 32 | 0.1650 |
| Mineral | 26 | 0.2199 |

Base (untuned) training results across all three algorithms, spatial R² / random R²:

| Target | RF | XGBoost | LightGBM |
|---|---|---|---|
| Fibric | 0.105 / 0.38 | 0.08 / 0.41 | **0.156 / 0.436** ← winner |
| Hemic | 0.137 / 0.31 | 0.060 / 0.43 | **0.216 / 0.446** ← winner |
| Sapric | **0.207 / 0.570** ← winner | 0.189 / 0.586 | 0.204 / 0.577 |
| Mineral | 0.216 / 0.631 | 0.218 / 0.640 | **0.293 / 0.641** ← winner |

LightGBM wins three of the four targets. Sapric is the exception where Random Forest
beats LightGBM by a small margin (0.207 vs. 0.204 spatial R²). The selection rule (best spatial R² per target) was applied across all four targets and Random Forest was kept as the Sapric model.

## 5.4 Post-Inference Normalization

Because each of the four targets is modeled independently, raw pixel-level
predictions do not sum to exactly 100%. A normalization step is applied after
inference to correct this:

```
stack = [Fibric_pred, Hemic_pred, Sapric_pred, Mineral_pred]   # per pixel
total  = sum(stack)
normed[i] = (stack[i] / total) * 100.0   for each component i
```

## 5.5 Outputs and Accuracy

### Composition surfaces

Each of the four component maps is finalized as a Cloud-Optimized GeoTIFF: 

- **Format:** `uint8`, where `pixel value × 0.5 = percent (0–100%)`
- **Nodata:** 255
- **Mask:** `PROB_LGBM_V2 ≥ 0.362` (same peatland extent as depth)

### Final model performance

| Target | Winning model | Spatial R² | Random R² | Random MAE |
|---|---|---|---|---|
| Fibric | COMP_LGBM_FIBRIC_PCT_TUNED | 0.156 | 0.436 | 11.9% |
| Hemic | COMP_LGBM_HEMIC_PCT_TUNED | 0.216 | 0.446 | 18.8% |
| Sapric | COMP_RF_SAPRIC_PCT_TUNED | 0.207 | 0.570 | 13.5% |
| Mineral | COMP_LGBM_MINERAL_PCT_TUNED | 0.293 | 0.641 | 16.2% |

### Interpreting low spatial R² values

Spatial R² for composition (0.16–0.29) is the lowest of the three models that were run, and the random versus spatial gap is large across all the targets. This is an expected result as organic decomposition state is largely impacted by local hydrology, water table depth, and local drainage that all vary at scales finer than the 50km spatial CV blocks used. A model can only generalize across a 50km region to the extent that decomposition state is explained by variables that also vary at that scale. The higher random CV R² values (0.44–0.64) confirm the models are learning real signals.
