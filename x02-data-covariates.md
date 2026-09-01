# 2 Data and Covariates

Each model in this project is created using two types of input. The first is the point
observations that are used to train and validate the models, and the second is a statewide covariate raster stack that is used to train models at each of the point locations as well as to create the spatial predictions that become the final raster map product. This section goes over where this data came from and how it was prepared.

## 2.1 Point Data

### 2.1.1 Data Sources

Training and validation observations are compiled from three primary sources:

- **Minnesota Peat Inventory** is a dataset collected by the Minnesota DNR during the 1970’s and 80’s. It contains probe measurement peat depths across peatlands in Minnesota as well as laboratory analyzed data including von post decomposition class, bulk density, carbon content, ash content, among other metrics at a horizon level. 
- **NASIS pedon data** is the USDA’s Natural Resource Conservation Services data set that populates soil survey. The dataset contains soil profile descriptions of both mineral and organic soils across the country 
- **Minnesota Biological Survey (MBS)** contributes presence and absence observations derived from native plant community classifications where peat associated community types were coded as peat present and upland or non peat communities were coded as peat absent. 

### 2.1.2 Curation by Model Type

The three data sources were combined and filtered depending on which
model the resulting dataset trains, producing three separate point CSVs:

| Dataset | Target | Current N |
|---|---|---|
| Probability | `peat_binary` (1/0) | 57,134 |
| Depth | `depb` (cm) | 8,223 |
| Composition | `Fibric/Hemic/Sapric/Mineral_pct` | 18,041 |

## 2.2 Covariate Stack

### 2.2.1 Computing Environment and Reference Grid

All covariate processing and model training was performed on the Minnesota Supercomputing Institute (MSI) Agate cluster (AMD EPYC, SLURM batch scheduling). A conda environment (gdalenvgeospat) was created to include geospatial libraries such as GDAL, rasterio, fiona, pyproj, shapely as well as machine learning packages scikit-learn, XGBoost, LightGBM, Optuna that were not previously available on MSI environments.

Every covariate is aligned to a reference grid (gNATSGO 10m MUKEY) using `gdalwarp` with bilinear resampling for the continuous covariates and nearest-neighbor for categorical covariates.

- **Dimensions:** 66,474 × 75,185 pixels
- **CRS:** EPSG:5070 (NAD83 / Conus Albers Equal Area)
- **Resolution:** 10 m
- **Origin:** (−99,098, 3,021,389)

The shared reference grid allows covariates coming from different sources to be stacked together and read pixel-for-pixel across the entire state boundary.

### 2.2.2 Terrain Derivatives

The USGS 3D Elevation Program (3DEP) 10m DEM was used as the elevation source. It was  mosaicked statewide and then clipped to the Minnesota state boundary. Terrain derivatives were computed from the DEM using WhiteboxTools (v2.4.0), using one of two processing methods depending on if the derivative depends on watershed context or not. 

- **Statewide, computed directly on the DEM**: slope, aspect, hillshade, plan/profile/mean/maximal curvature, geomorphons, Pennock landform class, deviation from mean elevation (4/8/16 m), and relative topographic position (4/8/16 m).
- **By HUC8 watershed** This method was used for derivatives that depend on water flow and are sensitive to upstream drainage. To accomplish this, Minnesota was split into 127 HUC8 watersheds, each buffered to 5km to avoid edge effects and processed independently tile by tile. Then each of the 127 tiles were mosaicked together with the buffer zones trimmed off to get full state coverage. This group includes the hydrologically conditioned (breached) DEM, D8 and D-infinity flow accumulation, and the topographic wetness index (TWI).

**Hydrological conditioning:** the DEM was hydrologically conditioned before
computing flow dependent derivatives. This was done using a five-step process on each HUC8 tile:

1. `FillSingleCellPits` — remove single-cell LiDAR noise artifacts
2. `FlattenLakes` — flatten lake surfaces to a uniform minimum elevation
3. `TopologicalBreachBurn` — burn the stream network into the DEM (snap tolerance: 2.0 m)
4. `BreachDepressionsLeastCost` — breach through road embankments and other
   artificial barriers (max breach length: 100 cells / 100 m; max cost: 1.0 m)
5. `FillSingleCellPits` — final cleanup

Input vector layers for this step were DNR HydroFeatures (lakes), DNR RiversStreams (streams), and MnDOT Roadway Routes (roads).

### 2.2.3 Imagery

**Sentinel-2** imagery (COPERNICUS/S2_SR_HARMONIZED, 2019–2024) was composited using Google Earth Engine (GEE) into three seasonal medians: spring (April–May), summer (June–August), and fall (September–October). Cloud masking was performed using the SCL layer (excluding cloud shadow, medium/high cloud, and thin cirrus classes) and a pre-filter dropping
images with >30% cloud cover. Spring and fall composites also masked snow and ice.
Each of the three seasonal composites contains 10 raw bands (B02–B08A, B11, B12) plus
two derived indices:
**NDVI** (B08−B04)/(B08+B04) shows vegetation health and density
**SWDI** (from green/B03 and red-edge-1/B05) shows soil moisture sensitivity

**Tasseled Cap** transformations (Brightness/TCB, Greenness/TCG, and Wetness/TCW) were
computed on MSI by applying coefficients to the Sentinel-2 bands. Outputs were checked against expected correlations (positive Brightness–Greenness correlation, negative Brightness–Wetness)

**Sentinel-1 SAR** data was composited from GEE. Each seasonal composite provides three bands: VV (dB), VH (dB), and the VV/VH ratio

### 2.2.4 Climate

**PRISM** 30-year climate normals were used for mean annual precipitation, mean July maximum temperature, mean annual temperature, and mean January minimum temperature. All four were resampled to 10m using bilinear interpolation.

### 2.2.5 Wetland and Soil Reference Layers

**National Wetlands Inventory (NWI):** Wetland polygons were rasterized and classified by Cowardin code into three groups: non-wetland, wetland, and lowland conifer (PFO4/PSS4/PFO2/PSS2 plus other codes with "/4"). Model experimentation found that the 3 class coding gave no improvement over a binary wetland/non-wetland coding, so the binary version was used as the final covariate.

**gNATSGO organic soils:** The gNATSGO raster was used to classify organic soils into 8 taxonomic classes as well as a binary "any organic component" layer. Both gNATSGO-derived products are peat classification layers and were excluded from the probability model training but retained as reference/validation layers. 

### 2.2.6 Distance Features

Euclidean distance rasters to water bodies, streams, and roads were computed from
Minnesota statewide vector layers (MnDOT Roadway Routes; DNR RiversStreams; DNR
HydroFeatures) using GDAL proximity tools.

### 2.2.7 Excluded Categorical Layers

Three one-hot encoded categorical layers: **quaternary geology classification**,
**Pennock landform class**, and **geomorphons** were tested early in the project
(03-peatland-carbon-modeling `exp401`–`exp408`) and eventually excluded from all
production models. These layers produced polygon shaped artifacts in the spatial inference outputs. 

## 2.3 Candidate Feature Set Summary

Combining all covariate groups above, the feature stack totaled 172 features before any filtering. Two reduction stages were then applied:

1. **Correlation filtering** (Pearson |r| ≥ 0.90, dropping the lower variance feature of
   each correlated pair) 
2. **Recursive Feature Elimination**, applied only for Random Forest models 

Final feature counts and the specific covariates retained are included in the model's results in Ch. 3 (Probability), Ch. 4 (Depth), and Ch. 5 (Composition).


## Sections to cover
- 2.1 Study area & peatland definition — extent, ecological provinces, peat/histosol definition, depth threshold
- 2.2 Reference/training data — NWI; gSSURGO/histosol layer; DNR landholdings & School Trust Lands; field obs and legacy cores/boreholes; provenance and vintages
- 2.3 Covariate stack — terrain/LiDAR derivatives, Sentinel-1, Sentinel-2, climate (PRISM), hydrography/ditch proximity; resolution 10m, projection, tiling scheme
- 2.4 Harmonization, QA/QC — resampling/alignment, gap-filling, cloud/water masking; Google Earth Engine + MSI HPC; storage; manifest-driven tiling; versioning
