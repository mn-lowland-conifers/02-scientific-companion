# 8. Aboveground Carbon Methods

Part II — Field Validation & Ecosystem Carbon (2025 Season)


## Sections to cover
- 8.1 Trees — inventory (species, DBH, height, status); allometric equations for black spruce and tamarack; biomass->carbon conversion
- 8.2 Understory???

1) Unit conversion: BASAL_AREA (ft²/ac) into m²/ha; stem density class (MN_DENS) into stems/ac using the class midpoint → stems/ha
2) QMD is derived per stand from basal area ÷ stem density
3) Species grouping: dominant species (MN_SPP) mapped to allometric group (Jenkins et al. 2003)
4) Above ground biomass: applied the Jenkins group equations to QMD (kg/tree)
5) Stand-level scaled: per-tree AGB × stems/ha results in units of kg/ha, then × 0.5 carbon (kgC/ha) and then kgC/m² (C_AGC_KGM2)
6) Filtering: drop stands that are year pre 2000 by setting carbon to nodata (-9999). This reduced the polygons by 33,796 stands. New field is C_AGC_V2
7) Age binning: AGE_BIN bins the stands by age 1 = 0–20yr, 2 = 21–80yr, 3 = 80+yr
8) Rasterization: I also have rasters made with the carbon number and age bin if you want those. 


Fields:
C_AGC_KGM2 - above-ground carbon (kgC/m²)
JENKINS_GR - Jenkins allometric species group
QMD_CM - quadratic mean diameter (cm) derived from basal area ÷ stem density
C_AGC_V2 - same as C_AGC_KGM2 but set to -9999 value for any SURVEY_YR < 2000
AGE_BIN - 1 = 0–20yr, 2 = 21–80yr, 3 = 80+yr
