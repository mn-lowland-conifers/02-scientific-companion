# 1 Introduction

## Purpose

This document is the scientific companion to the Lowland Conifer Legislative report. It goes over the findings of this project and how those findings compare to and are supported by existing peatland carbon literature, and what it means for management of peatlands on Minnesota School Trust and other lands.

- Ch. 2 — Data and Covariates
- Ch. 3 — Peatland Probability Model
- Ch. 4 — Peat Depth Model
- Ch. 5 — Organic Decomposition (Composition) Model
- Ch. 6 — Carbon Stock Estimation
- Ch. 7 — Field Campaign Design
- Ch. 8 — Above-Ground Carbon Methods
- Ch. 9 — Below-Ground Carbon Methods
- Ch. 10 — Validation Analysis
- Ch. 11 — Systematic Review Protocol
- Ch. 12 — Meta-Analytic Methods
- Ch. 13 — Results by Theme
- Ch. 14 — Evidence Map and Gaps
- Ch. 15 — Linking Stocks to Management
- Ch. 16 — Limitations and Uncertainty
- Ch. 17 — Conclusions
- Ch. 18 — Data and Code Availability
- Ch. 19 — References

### 1.1 Northern peatland carbon — global stocks, belowground dominance (Gorham 1991; Yu 2012; Bridgham et al. 2006)
### 1.2 Lowland conifer bog & fen systems — black spruce / tamarack ecology; ombrotrophic-minerotrophic gradient; MN peatland landscape and extent
### 1.3 Ecosystem carbon architecture — peat >> tree biomass; pool definitions (above-/belowground; acrotelm/catotelm) (Beaulne et al. 2021)
### 1.4 Disturbance regimes & dynamics — harvest, fire, insect/disease; paludification feedback
### 1.5 Digital soil mapping of peatland carbon — state of the art; what's missing (decomposition-state mapping; statewide validated stocks)
### 1.6 Objectives — (i) map statewide extent/depth/decomposition/carbon; (ii) validate against 2025 field campaign; (iii) synthesize management science; (iv) integrate stocks with management sensitivity

## Other peatland mapping projects and literature

| Project | Who | Scope | Goals & Findings | Acreage/Carbon | Link |
|---|---|---|---|---|---|
| Minnesota Peat Inventory | MN DNR peat program (1976–1982) | Field inventory of 20 counties in northern MN | Program was implented to asses peat as an energy source during the 1970s energy crisis. Goals were (1) horticultural potential (2) energy potential (3) informing statewide peatland management policy. Measured peat type, depth, and decomposition at sites in northern MN. | Data exists in the database, county reports, and a shapefile that contains the point data.  | https://gis.data.mn.gov/datasets/173d3993a9f2433bb093c970ac3e8d38_0/about |
| TNC Peatland Playbook | The Nature Conservancy (Jan 2025) | Outlines approaches to protect and restore peatlands as a climate solution | Overview of peatland and the science behind peatlands with focus on peatland restorations for drained and ditched peatlands. Established from a 2024 TNC effort to build a new statewide restoration strategy | Crosswalked NWI, SSURGO, and NLCD for layer that shows >10% histosol content (7.8 million acres) estimated carbon stocks with SSURGO at 4.49 billion metric tons | https://www.nature.org/content/dam/tnc/nature/en/documents/PeatlandPlaybook-Jan25.pdf |
| "The Potential for Terrestrial Carbon Sequestration in Minnesota" (2008 LCCMR) | Hobbie, Nater & Reich (UMN), commissioned by MN Legislature (2008) | Statewide synthesis/literature review of carbon sequstration in Minnesota | Overall look at how much carbon MN lands (forests, peat, prairie, cropland) could sequester or lose. Focuses on landcover type, sequestration type, and sequestration type change. The recommendataions were to (1) preserve existing large C stocks in peatlands and forests (2) prioritize land-use changes most likely to sequester C and (3) invest in long term monitoring networks. | Estimates 5.73 million acres of peatland (Peat inventory and NASIS) and 4,250 megatonnes of carbon (745 metric tonnes of stored C per acre) | https://files.dnr.state.mn.us/aboutdnr/reports/carbon2008.pdf |
| BWSR Peatland Mapping Tool | MN Board of Water & Soil Resources | State agency program and GIS mapping tool | Summary of the programs and funding for peatland restoration. Identifies drained/partially drained peatlands for restoration. | Cites ~7 million acres from TNC. Includes arcgis tool that shows peatlands (potentially wet histosol layer) as well as easements, state trust lands, and ditches | https://bwsr.state.mn.us/peatlands |
| DNR Lowland Conifer Old Growth (LCOG) | MN DNR | Lowland conifer old growht evaluation  | To identify and protect lowland conifer old growth forest on DNR lands. The original 2003 old-growth designation deferred lowland conifer because complexity and data gaps made it hard to define old growth characteristics. Identified 41,000 acres of new canidate stands in 2021. 27,000 acres on school trust land |  | https://www.dnr.state.mn.us/input/mgmtplans/lcog.html |
| Marcell Experimental Forest research | USFS Northern Research Station, Randy Kolka | Long-term field research (1960 to present) | Long running peatland research with many papers as a result. Include topics such as peatland carbon cycling, water flow, carbon dioxide and methane fluxes. Relevant findings: surface peat releases carbon under warming, but deep peat carbon is comparatively stable. Hosts the SPRUCE experiment. | Peatlands make up of 3% of Earth's land surface and 30% of global soil carbon | https://research.fs.usda.gov/nrs/forestsandranges/locations/marcell |
| SPRUCE (Spruce and Peatland Responses Under Changing Environments) | USFS, DOE, ORNL | Experiments on terrestrial ecosystems response to warming and elevated CO2 | Ten enclosures on a bog in northern MN with differing raised air and soil temperatures to simulate future climate scenarios on peat. Findings so far: warming increases surface decomposition and methane/CO2 release and reduces soil biodiversity. Deep peat carbon remains relatively stable in the near term. |  | https://research.fs.usda.gov/nrs/projects/spruce |
| "Mapping peatland extent and condition in the conterminous United States and Hawaii..." | Lilleskov, McCullough & Uhelski (USFS Northern Research Station), Journal of Environmental Management, 2025 | National peatland mapping | National map of peatland extent and condition for the Lower 48 + Hawaii. Combines soil survey data (histosols/histic epipedons), land-use, ditch, road, and railroad proximity, protection status, and NRCS easement layers. Feeds into USFS's  PeatRestore project. Estimated GHG emissions as well and how restoration on drained histosols can reduce national GHG emissions.  | Analyzed 94,750 km2 of histosols and 13,533 km2 of histic epipedons. 7% of the histosols were in agricultural usea and 19% of non-agricultural peatlands were within 150m of a ditch/road/rail. Estimated potential emissions reduction from rewetting currently drained peatlands >36.8 Tg CO2-eq/year. | https://www.sciencedirect.com/science/article/abs/pii/S0301479725030762 |

