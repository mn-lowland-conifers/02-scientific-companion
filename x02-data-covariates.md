# 2. Data & Covariates

Each model in this project is created using two types of input. The first is the point observations used to train and validate the models. The second is a statewide covariate raster stack, which is sampled at each point location to train the models and is used to create the spatial predictions that become the final raster map products. This section summarizes the process of data consolidation and curation. Additional details on processing parameters, software, and implementation are documented in the technical ebook (03-peatland-carbon-modeling).

## 2.1 Study Area and Peatland Definition

The study area of this project is the entire state of Minnesota. Peat occurrence is treated as a spatial prediction target and is not synonymous with wetlands or mapped organic soils. Peat presence was defined from the point data observations described below and represented with the binary target `peat_binary`, where 1 denotes peat present and 0 denotes peat absent. Peat probability, peat depth, and peat decomposition are modeled as separate targets.

## 2.2 Reference and Training Data

### 2.2.1 Point Observations

Training and validation observations were compiled from three primary sources.

**Minnesota Peat Inventory** is a dataset collected by the Minnesota DNR during the 1970s and 1980s. It contains probe measurement peat depths across peatlands in Minnesota as well as laboratory analyzed data including von post decomposition class, bulk density, carbon content, ash content, among other metrics at a horizon level. 


**NASIS pedon data** The National Soil Information System (NASIS) is the USDA Natural Resources Conservation Service database that populates soil survey. The dataset contains soil profile descriptions of both mineral and organic soils across the country.

**Minnesota Biological Survey Relevé data** The MBS relevé dataset contributes presence and absence observations derived from native plant community classifications. Peat associated community types are coded as peat present and non-peat (upland) communities are coded as peat absent. Because these labels are inferred from plant community type and not directly measured they only populate peat occurrence.

The three sources were combined and filtered according to the model the resulting dataset trains, producing three separate point datasets:

**Point datasets used to train and validate each model.**

| Dataset | Target | N |
|---|---|---|
| Probability | `peat_binary` (1/0) | 57,134 |
| Depth | `depb` (cm) | 8,223 |
| Composition | `Fibric_pct`, `Hemic_pct`, `Sapric_pct` | 18,041 |

### 2.2.2 Wetland and Soil Reference Layers

Two statewide layers provide wetland and soil context. The National Wetlands Inventory (NWI) was rasterized and classified by Cowardin code into three groups: non-wetland, wetland, and lowland conifer. Model experiments found that this three-class coding gave no improvement over a binary wetland/non-wetland coding, so the binary version was used as a covariate. The gNATSGO raster was used to classify organic soils into eight taxonomic classes, as well as a binary "any organic component" layer. Both gNATSGO-derived products are peat classification layers, so they were excluded from probability model training and retained as reference and validation layers.

## 2.3 Covariate Stack

Environmental covariates representing terrain, hydrology, vegetation, climate, and landscape were curated as spatial raster data. All covariates were aligned to a 10m grid in EPSG:5070 (NAD83 / Conus Albers Equal Area) which allows layers from different sources to be stacked and read pixel-for-pixel. In total, there were 172 candidate features before any filtering was completed.

**Terrain and hydrology.** The USGS 3D Elevation Program (3DEP) 10m digital elevation model (DEM) was mosaicked statewide and used to compute terrain derivatives with WhiteboxTools. This included derivatives such as slope, aspect, hillshade, curvature, deviation from mean elevation, and relative topographic position. Derivatives that are impacted by water flow were computed separately by HUC8 watershed, with a 5km buffer applied around each HUC8 boundary to avoid edge effects and then mosaicked together. The DEM was first hydrologically conditioned then derivatives such as D8 and D-infinity flow accumulation and topographic wetness index (TWI) were calculated.

**Imagery.** Sentinel-2 surface reflectance imagery (2019–2024) was composited in Google Earth Engine (GEE) into spring (April–May), summer (June–August), and fall (September–October) medians after cloud masking, with snow and ice also masked in spring and fall. Each seasonal composite contains ten spectral bands and two derived indices: the normalized difference vegetation index (NDVI), which indicates vegetation health and density, and SWDI, derived from the green and red-edge-1 bands, which indicates soil moisture sensitivity. Tasseled Cap Brightness, Greenness, and Wetness components were also computed from the Sentinel-2 imagery using the coefficients of Shi and Xu (2019).

**Radar imagery.** Sentinel-1 synthetic aperture radar (SAR) data was composited in GEE. Each seasonal composite includes VV backscatter (dB), VH backscatter (dB), and the VV/VH ratio.

**Climate.** PRISM 30-year climate normals were used for mean annual precipitation, mean July maximum temperature, mean annual temperature, and mean January minimum temperature. All layers and were resampled to 10m. Because of their coarseness these layers represent large climate gradients and not local variations.

**Distance features.** Euclidean distance rasters to water bodies, streams, and roads were computed from state vector layers.


## 2.4 Harmonization and QC

Each covariate was aligned to a reference grid (gNATSGO 10m MUKEY raster) using bilinear resampling for continuous covariates and nearest-neighbor resampling for categorical covariates. Quality control steps were taken with the imagery layers including screening Sentinel-2 imagery with scene-level cloud thresholds and pixel-level masks, and verifying the Tasseled Cap outputs against expected correlation values (a positive Brightness–Greenness correlation and a negative Brightness–Wetness correlation).

The covariate raster stack was reduced in two stages before modeling. First, correlation filtering (Pearson |r| ≥ 0.90) removed the lower-variance feature of each correlated pair. Second, recursive feature elimination (RFE) was applied to the Random Forest models only. The retained covariates differ depending on model and target, and they are reported with each model.

Imagery compositing was performed in GEE. Terrain processing, covariate alignment, and model training were performed on the Minnesota Supercomputing Institute (MSI) Agate cluster. The software environment, tiling scheme, file organization, and processing parameters are documented in the technical methods ebook (03-peatland-carbon-modeling).

## References

Shi, T., and Xu, H. (2019). Derivation of Tasseled Cap Transformation coefficients for Sentinel-2 MSI at-sensor reflectance data. *IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing*, 12(10), 4038–4048.
