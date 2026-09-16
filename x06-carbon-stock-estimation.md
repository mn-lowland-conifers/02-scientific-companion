# 6. Peat Carbon Stock Estimation

Part I — Statewide Peatland Carbon: Digital Soil Mapping


## Sections to cover
- 6.1 Stock formulation — C stock = sum(depth increment x bulk density x %C), modulated by decomposition state; above- vs belowground accounting;
- 6.2 Bulk density & %C — measured relationships / pedotransfer functions; dependence on decomposition state and peat type
- 6.3 Upscaling & aggregation — pixel -> landscape; aggregation by ownership, county, ecological province, peatland type, condition (intact/drained/degraded)
- 6.4 Uncertainty propagation — Monte Carlo / error-stacking across probability, depth, BD, %C, decomposition layers; reported as estimate +/- interval

# 6. Peat Carbon Stock Estimation

Part I — Statewide Belowground Carbon: Digital Soil Mapping

## 6.1 Full-Profile Carbon Stock — Depth-Curve Method

The statewide full profile carbon stock map is made with a depth curve model rather
than using the modeling approach similar to the probability and depth workflows.

**Training set:** data from the Minnesota Peat inventory was used to pull the 1,609 criteria B pedons (§9.4) which were profiles that have an observed non organic bottom along with complete bulk density and SOC% lab data (measured or gap-filled, §9.3).

**Models** Two candidate models, a linear and random forest, were fit using the criteria B data set and maximum observed peat depth as the predictor. Both were evaluated on 50km block spatial cross-validation and with random cross-validation (5 folds each). The linear model was selected as the final based on its spatial CV performance (§6.1, table).

The linear model was then refitted using the full training set of all 1,609 criteria B pedons together, not just a single CV fold, to produce the coefficients used. That final fit is:

> C_stock_full_kgm2 = 0.5043 × max_peat_depth_cm + 20.12

Where 0.5043 kgC/m² per cm of depth is the fitted slope and 20.12 kg C/m² is the fitted intercept. This means that for every additional centimeter of peat depth around half a kilogram of carbon per square meter is gained on top of the baseline of 20 kgC/m² at zero depth (the
intercept). This equation is then applied to the already created depth raster to get a spatial inference for the entire state. 

The cross validation metrics below show how well the linear regression on depth generalizes to pedons that were held out of the training data, which was computed from out-of-fold predictions during the model-comparison step.

| Model | Spatial R² | Spatial MAE | Random R² |
|---|---|---|---|
| **Linear (final)** | **0.8623** | **16.14** | 0.8673 |
| RF (depth-only) | 0.8456 | 16.83 | 0.8506 |

The spatial and random CV R² values are much closer for the linear model (0.862 vs.
0.867) compared to the probability and depth models (Ch. 3–4) where the spatial CV was much lower than the random CV. This is expected in this case because depth is a direct driver of the carbon stock (more peat means more carbon) rather than an indirect indicator based on covariate clustering.

**Statewide application:** the fitted depth-curve was applied to the statewide depth raster masked to the probability threshold (≥ 0.362)
