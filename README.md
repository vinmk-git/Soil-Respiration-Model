# Forest Soil Respiration Model Under Climate Warming

In this project, I am modeling annual soil respiration rates based on soil moisture and surface temperature projections.
The model predicts forest soil CO₂ release across tropical, subtropical, temperate, and boreal biomes until 2099 under three CMIP6 climate scenarios – SPP1-2.6, SPP2-4.5, and SPP5-8.5 – the same data underlying the IPCC policy reports.

This model is built in Python on historical soil moisture data as well as data from the Soil Respiration Database (SRDB). 

This is the second project in a portfolio I'm building as part of my transition from nanotechnology into soil biogeochemistry. The first project is an [interactive CO₂ emissions dashboard](https://github.com/vinmk-git/CO2-Emissions-Dashboard) built in R Shiny.

---

## What this project does

Soil respiration is the release of CO₂ from soils via microbial decomposition and root activity, and it is one of the largest carbon fluxes in the terrestrial carbon cycle, and one of the bigger uncertainties in how ecosystems will respond to warming.
This project, therefore, fits a Q10-based exponential respiration model to observational forest soil respiration data from the [Soil Respiration Database (SRDB)](https://github.com/bpbond/srdb). [1] The model can additionally also include historical soil moisture measurements using a quadratic model – a relationship that is well-documented in the literature. [2-4]
I then project respiration rates forward to 2099 using temperature and soil moisture projection data from the [Copernicus Climate Data Store (CDS)](https://cds.climate.copernicus.eu/).

The model is benchmarked against a Random Forest regressor, evaluated via stratified cross-validation, and visualized interactively on global maps using hvplot.

---

## Methods

### Data
- **SRDB** (v20250503): Annual soil respiration rates, mean annual temperature, biome classification, and site coordinates for global forest measurement sites
- **CMIP6 / CDS**: Monthly near-surface air temperature (`tas`) and top-layer soil moisture (`mrsos`) under three SSP scenarios:
  - SSP1-2.6 (low emissions – equivalent to up to 1.8°C warming by 2100)
  - SSP2-4.5 (intermediate – 3.5 °C)
  - SSP5-8.5 (high emissions – 5.7 °C)
- Historical CDS moisture data used to match SRDB sites to observed moisture conditions at the time of measurement

### Model

Soil respiration is modelled as an exponential function of temperature (Q10 formulation), with an optional soil moisture correction:

**Temperature-only (T):**
```
Rs = R_base × Q10^((T - T_base) / 10)
R_base - base soil respiration rate
T_base – base temperature
```

**Temperature (T) + moisture (M):**
```
Rs = R_base × Q10^((T - T_base) / 10) - (α·M) + (β·M²) + γ + δ·(T·M)
```

Parameters are fitted by minimizing mean squared error. Model performance is evaluated via stratified k-fold cross-validation stratified by biome, and benchmarked against a Random Forest regressor.

### Spatial matching

SRDB site coordinates are matched to the nearest CMIP6 grid cell using a KDTree nearest-neighbor search, linking each observational record to a projected climate time series for 2015–2099.

---

## Key findings

*Full results pending completion of all SSP scenario runs.*

Preliminary results suggest soil respiration increases across all biomes under warming, with the strongest absolute increases in tropical forests and the strongest relative increases in boreal systems — consistent with the literature on Q10 latitudinal gradients.

---

## Repository structure
Due to their size, the zip files containing the climate data need to be downloaded separately.
A Google Colab link to a functional notebook with the entire structure is provided, however.

```
├── soilRmodel.ipynb                   # Main analysis notebook
├── fetching_CDS_data.ipynb            # CDS data download and API setup
├── historical_data_moisture.zip
├── ssp126_data.zip / ssp245_data.zip / ssp585_data.zip 
├── ssp126_data_temperature.zip / ssp245_data_temperature.zip / ssp585_data_temperature.zip
└── srdb-20250503a/
    ├── srdb-data.csv
    └── srdb-equations.csv
```

---

## Requirements

```
python >= 3.9
pandas, numpy, matplotlib, xarray, scipy, scikit-learn
holoviews, geoviews, hvplot, plotly, bokeh, h5netcdf, modsim
```

```bash
pip install pandas numpy matplotlib xarray scipy scikit-learn holoviews geoviews hvplot plotly bokeh h5netcdf modsim
```

### Data access

Climate data must be downloaded separately from the Copernicus Climate Data Store — see `fetching_CDS_data.ipynb` for setup (CDS account and API key required).

SRDB data is available at https://github.com/bpbond/srdb.

---

## Usage

1. Sign up in Copernicus Climate Data Store (CDS) to obtain an API key.
2. Download climate data using `fetching_CDS_data.ipynb`
3. Place SRDB files in `srdb-20250503a/`
4. Run `soilRmodel.ipynb` top to bottom
5. Change the `scenario` variable near the top to switch between `"ssp1_2_6"`, `"ssp2_4_5"`, and `"ssp5_8_5"`

---

## A note on AI use

I used Claude (Anthropic) during this project for debugging, helping with writing this README, and code review.
The modelling decisions, data choices, and scientific interpretation are my own.

---

## About

**vin ilegueuno kadiri** — PhD in Chemistry / Nanotechnology, Max Planck Institute for Intelligent Systems.
Currently transitioning into soil biogeochemistry and building toward a postdoc in plant-soil-microbiome interactions.

[vinilegueuno.com](https://vinilegueuno.com) · [LinkedIn](https://www.linkedin.com/in/vinilegueunokadiri/)

---

## References
[1] Meyer, N., Welp, G. and Amelung, W., 2018. The temperature sensitivity (Q10) of soil respiration: controlling factors and spatial prediction at regional scale based on environmental soil classes. Global Biogeochemical Cycles, 32(2), pp.306-323.

[2] Nie, C., Li, Y., Niu, L., Liu, Y., Shao, R., Xu, X. and Tian, Y., 2019. Soil respiration and its Q10 response to various grazing systems of a typical steppe in Inner Mongolia, China. PeerJ, 7, p.e7112.

[3] Yu, H., Xu, Z., Zhou, G. and Shi, Y., 2020. Soil carbon release responses to long-term versus short-term climatic warming in an arid ecosystem. Biogeosciences, 17(3), pp.781-792.

[4] Cui, Y.B., Feng, J.G., Liao, L.G., Yu, R., Zhang, X., Liu, Y.H., Yang, L.Y., Zhao, J.F. and Tan, Z.H., 2020. Controls of temporal variations on soil respiration in a tropical lowland rainforest in Hainan Island, China. Tropical Conservation Science, 13, p.1940082920914902.
