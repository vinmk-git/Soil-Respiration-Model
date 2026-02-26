# Forest Soil Respiration Model Under Climate Warming

A process-based model of forest soil CO₂ release across tropical, subtropical, temperate, and boreal biomes — projected to 2099 under three CMIP6 climate scenarios. In this project, I am modeling annual soil respiration rates based on IPCC soil moisture and temperature predictions (from the Copernicus Climate Data Store).
The model is built in Python on historical soil moisture data as well as the Soil Respiration Database (SRDB). 

This is the second project in a portfolio I'm building as part of my transition from nanotechnology into soil biogeochemistry. The first project is an [interactive CO₂ emissions dashboard](https://github.com/vinmk-git/CO2-Emissions-Dashboard) built in R/Shiny.

---

## What this project does

Soil respiration — the release of CO₂ from soils via microbial decomposition and root activity — is one of the largest carbon fluxes in the terrestrial carbon cycle, and one of the bigger uncertainties in how ecosystems will respond to warming. This project fits a Q10-based respiration model to observational forest data from the [Soil Respiration Database (SRDB)](https://github.com/bpbond/srdb), then projects respiration rates forward to 2099 using temperature and soil moisture data from the [Copernicus Climate Data Store (CDS)](https://cds.climate.copernicus.eu/).

The model is benchmarked against a Random Forest regressor, evaluated via stratified cross-validation, and visualized interactively on global maps.

---

## Methods

### Data
- **SRDB** (v20250503): Annual soil respiration rates, mean annual temperature, biome classification, and site coordinates for global forest measurement sites
- **CMIP6 / CDS**: Monthly near-surface air temperature (`tas`) and top-layer soil moisture (`mrsos`) under three SSP scenarios:
  - SSP1-2.6 (low emissions)
  - SSP2-4.5 (intermediate)
  - SSP5-8.5 (high emissions)
- Historical CDS moisture data used to match SRDB sites to observed moisture conditions at the time of measurement

### Model

Soil respiration is modelled as an exponential function of temperature (Q10 formulation), with an optional soil moisture correction:

**Temperature-only:**
```
Rs = R_base × Q10^((T - T_base) / 10)
```

**Temperature + moisture:**
```
Rs = R_base × Q10^((T - T_base) / 10) × f(M)
where f(M) = (-α·M) - (β·M²) - γ - δ·(T·M)
```

Parameters are fitted by minimizing mean squared error using L-BFGS-B optimization. Model performance is evaluated via stratified 10-fold cross-validation stratified by biome, and benchmarked against a Random Forest regressor.

### Spatial matching

SRDB site coordinates are matched to the nearest CMIP6 grid cell using a KDTree nearest-neighbor search, linking each observational record to a projected climate time series for 2015–2099.

---

## Key findings

*Full results pending completion of all SSP scenario runs.*

Preliminary results suggest soil respiration increases across all biomes under warming, with the strongest absolute increases in tropical forests and the strongest relative increases in boreal systems — consistent with the literature on Q10 latitudinal gradients (Bekku et al., 2003; Zhou et al., 2009).

---

## Repository structure

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

Climate data must be downloaded separately from the Copernicus Climate Data Store — see `fetching_CDS_data.ipynb` for setup (CDS account and API key required). SRDB data is available at https://github.com/bpbond/srdb.

---

## Usage

1. Download climate data using `fetching_CDS_data.ipynb`
2. Place SRDB files in `srdb-20250503a/`
3. Run `soilRmodel.ipynb` top to bottom
4. Change the `scenario` variable near the top to switch between `"ssp1_2_6"`, `"ssp2_4_5"`, and `"ssp5_8_5"`

---

## A note on AI use

I used Claude (Anthropic) during this project for debugging, writing this README, and code review. All modelling decisions, data choices, and scientific interpretation are my own.

---

## About

**vin ilegueuno kadiri** — PhD in Chemistry / Nanotechnology, Max Planck Institute for Intelligent Systems. Currently transitioning into soil biogeochemistry and building toward a postdoc in plant-soil-microbiome interactions.

[vinilegueuno.com](https://vinilegueuno.com) · [LinkedIn](https://www.linkedin.com/in/vincentmkadiri/)
